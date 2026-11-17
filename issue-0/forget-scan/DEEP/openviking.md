# openviking: B3 static correctness review

Snapshot `e855796c83f9eb575d9b0f3082926660730f6a63` (MANIFEST.json row `openviking`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`, in-process HTTP adapter to OpenViking; local lifecycle helpers can provision/start a separate server. The adapter itself is not the vector engine (`sources/openviking/examples/hermes-plugin/__init__.py:3349`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/openviking/examples/hermes-plugin/__init__.py:3408` | `sources/openviking/examples/hermes-plugin/__init__.py:3410` |
| `is_available` | `sources/openviking/examples/hermes-plugin/__init__.py:1540` | `sources/openviking/examples/hermes-plugin/__init__.py:1542` |
| `get_config_schema` | `sources/openviking/examples/hermes-plugin/__init__.py:1552` | `sources/openviking/examples/hermes-plugin/__init__.py:1553` |
| `save_config` | `sources/openviking/examples/hermes-plugin/__init__.py:1555` | `sources/openviking/examples/hermes-plugin/__init__.py:1557` |
| `initialize` | `sources/openviking/examples/hermes-plugin/__init__.py:1760` | `sources/openviking/examples/hermes-plugin/__init__.py:1761` |
| `on_turn_start` | `sources/openviking/examples/hermes-plugin/__init__.py:1965` | `sources/openviking/examples/hermes-plugin/__init__.py:1966` |
| `system_prompt_block` | `sources/openviking/examples/hermes-plugin/__init__.py:1972` | `sources/openviking/examples/hermes-plugin/__init__.py:1973` |
| `prefetch` | `sources/openviking/examples/hermes-plugin/__init__.py:2002` | `sources/openviking/examples/hermes-plugin/__init__.py:2004` |
| `queue_prefetch` | `sources/openviking/examples/hermes-plugin/__init__.py:2014` | `sources/openviking/examples/hermes-plugin/__init__.py:2016` |
| `sync_turn` | `sources/openviking/examples/hermes-plugin/__init__.py:2589` | `sources/openviking/examples/hermes-plugin/__init__.py:2593` |
| `on_session_end` | `sources/openviking/examples/hermes-plugin/__init__.py:3020` | `sources/openviking/examples/hermes-plugin/__init__.py:3023` |
| `on_session_switch` | `sources/openviking/examples/hermes-plugin/__init__.py:3040` | `sources/openviking/examples/hermes-plugin/__init__.py:3052` |
| `on_memory_write` | `sources/openviking/examples/hermes-plugin/__init__.py:3126` | `sources/openviking/examples/hermes-plugin/__init__.py:3134` |
| `get_tool_schemas` | `sources/openviking/examples/hermes-plugin/__init__.py:3161` | `sources/openviking/examples/hermes-plugin/__init__.py:3162` |
| `handle_tool_call` | `sources/openviking/examples/hermes-plugin/__init__.py:3164` | `sources/openviking/examples/hermes-plugin/__init__.py:3165` |
| `shutdown` | `sources/openviking/examples/hermes-plugin/__init__.py:3175` | `sources/openviking/examples/hermes-plugin/__init__.py:3179` |
| `shutdown` | `sources/openviking/examples/hermes-plugin/native_memory_mirror.py:122` | `sources/openviking/examples/hermes-plugin/native_memory_mirror.py:124` |

## 2. Forget paths and derived copies

`yes`: `viking_forget` validates one concrete own-user `.md` URI, rejects generated summaries and broad paths, then DELETE `/api/v1/fs` with `recursive=False` (`sources/openviking/examples/hermes-plugin/__init__.py:3346`; `sources/openviking/examples/hermes-plugin/__init__.py:3349`; `sources/openviking/examples/hermes-plugin/__init__.py:645`). Engine `fs_service.rm` delegates to `_rm`, which removes filesystem data, synchronizes watches, schedules semantic refresh, cleans resource-memory links, and refreshes memory overviews (`sources/openviking/openviking/service/fs_service.py:498`). `partial`: refresh can be deferred and independent session archives/source text are not all erased by deleting one memory file (`sources/openviking/openviking/service/fs_service.py:509`; `sources/openviking/openviking/service/fs_service.py:545`). Other engine removal/session/admin routes are in the inventory.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/base_client.rs:573`; body evidence.

- `sources/openviking/crates/ov_cli/src/base_client.rs:572` ``
- `sources/openviking/crates/ov_cli/src/base_client.rs:573` `pub async fn delete<T: DeserializeOwned + 'static>(`
- `sources/openviking/crates/ov_cli/src/base_client.rs:574` `&self,`
- `sources/openviking/crates/ov_cli/src/base_client.rs:575` `path: &str,`
- `sources/openviking/crates/ov_cli/src/base_client.rs:576` `params: &[(String, String)],`
- `sources/openviking/crates/ov_cli/src/base_client.rs:577` `) -> Result<T> {`
- `sources/openviking/crates/ov_cli/src/base_client.rs:581` `.http`
- `sources/openviking/crates/ov_cli/src/base_client.rs:582` `.delete(&url)`
- `sources/openviking/crates/ov_cli/src/base_client.rs:583` `.headers(self.build_headers())`
- `sources/openviking/crates/ov_cli/src/base_client.rs:584` `.query(&params);`
- `sources/openviking/crates/ov_cli/src/base_client.rs:585` `let response = self.send_request(request, "HTTP request failed").await?;`
- `sources/openviking/crates/ov_cli/src/base_client.rs:586` ``
- `sources/openviking/crates/ov_cli/src/base_client.rs:589` ``
- `sources/openviking/crates/ov_cli/src/base_client.rs:590` `pub async fn delete_with_body<B: serde::Serialize, T: DeserializeOwned + 'static>(`
- `sources/openviking/crates/ov_cli/src/base_client.rs:591` `&self,`
- `sources/openviking/crates/ov_cli/src/base_client.rs:592` `path: &str,`
- `sources/openviking/crates/ov_cli/src/base_client.rs:593` `body: &B,`
- `sources/openviking/crates/ov_cli/src/base_client.rs:594` `) -> Result<T> {`
- `sources/openviking/crates/ov_cli/src/base_client.rs:597` `.http`
- `sources/openviking/crates/ov_cli/src/base_client.rs:598` `.delete(&url)`
- `sources/openviking/crates/ov_cli/src/base_client.rs:599` `.headers(self.build_headers())`
- `sources/openviking/crates/ov_cli/src/base_client.rs:600` `.json(body);`
- `sources/openviking/crates/ov_cli/src/base_client.rs:601` `let request = if self.profile_enabled {`
- `sources/openviking/crates/ov_cli/src/base_client.rs:602` `request.query(&[("profile", "1")])`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/client.rs:255`; body evidence.

- `sources/openviking/crates/ov_cli/src/client.rs:254` ``
- `sources/openviking/crates/ov_cli/src/client.rs:255` `pub async fn delete<T: DeserializeOwned + 'static>(`
- `sources/openviking/crates/ov_cli/src/client.rs:256` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:257` `path: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:258` `params: &[(String, String)],`
- `sources/openviking/crates/ov_cli/src/client.rs:259` `) -> Result<T> {`
- `sources/openviking/crates/ov_cli/src/client.rs:260` `self.base.delete(path, params).await`
- `sources/openviking/crates/ov_cli/src/client.rs:261` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:262` ``
- `sources/openviking/crates/ov_cli/src/client.rs:263` `pub async fn delete_with_body<B: serde::Serialize, T: DeserializeOwned + 'static>(`
- `sources/openviking/crates/ov_cli/src/client.rs:264` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:265` `path: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:266` `body: &B,`
- `sources/openviking/crates/ov_cli/src/client.rs:267` `) -> Result<T> {`
- `sources/openviking/crates/ov_cli/src/client.rs:268` `self.base.delete_with_body(path, body).await`
- `sources/openviking/crates/ov_cli/src/client.rs:269` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:270` ``
- `sources/openviking/crates/ov_cli/src/client.rs:271` `pub async fn patch<B: serde::Serialize, T: DeserializeOwned + 'static>(`
- `sources/openviking/crates/ov_cli/src/client.rs:272` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:454` ``
- `sources/openviking/crates/ov_cli/src/client.rs:455` `pub async fn acl_delete(&self, uri: &str) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:456` `self.delete("/api/v1/acl", &[("uri".to_string(), uri.to_string())])`
- `sources/openviking/crates/ov_cli/src/client.rs:457` `.await`
- `sources/openviking/crates/ov_cli/src/client.rs:458` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:459` ``
- `sources/openviking/crates/ov_cli/src/client.rs:460` `fn build_write_body(`
- `sources/openviking/crates/ov_cli/src/client.rs:688` `];`
- `sources/openviking/crates/ov_cli/src/client.rs:689` `self.delete("/api/v1/fs", &params).await`
- `sources/openviking/crates/ov_cli/src/client.rs:690` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:691` ``
- `sources/openviking/crates/ov_cli/src/client.rs:692` `pub async fn mv(&self, from_uri: &str, to_uri: &str) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:693` `let body = serde_json::json!({`
- `sources/openviking/crates/ov_cli/src/client.rs:1060` `if path.is_dir() || path.is_file() {`
- `sources/openviking/crates/ov_cli/src/client.rs:1061` `let archive = if path.is_dir() {`
- `sources/openviking/crates/ov_cli/src/client.rs:1062` `Some(if show_progress {`
- `sources/openviking/crates/ov_cli/src/client.rs:1063` `self.zip_directory_with_progress(path, verbose, None)?`
- `sources/openviking/crates/ov_cli/src/client.rs:1064` `} else {`
- `sources/openviking/crates/ov_cli/src/client.rs:1065` `self.zip_directory(path, None)?`
- `sources/openviking/crates/ov_cli/src/client.rs:1069` `};`
- `sources/openviking/crates/ov_cli/src/client.rs:1070` `let upload = archive.as_ref().map(|file| file.path()).unwrap_or(path);`
- `sources/openviking/crates/ov_cli/src/client.rs:1071` `let id = if show_progress {`
- `sources/openviking/crates/ov_cli/src/client.rs:1072` `self.upload_temp_file_with_progress(upload, verbose).await?`
- `sources/openviking/crates/ov_cli/src/client.rs:1073` `} else {`
- `sources/openviking/crates/ov_cli/src/client.rs:1074` `self.upload_temp_file(upload).await?`
- `sources/openviking/crates/ov_cli/src/client.rs:1287` `};`
- `sources/openviking/crates/ov_cli/src/client.rs:1288` `self.delete(&path, &params).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1289` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1290` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1291` `// ============ Task Methods ============`
- `sources/openviking/crates/ov_cli/src/client.rs:1292` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1504` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1505` `pub async fn admin_delete_account(&self, account_id: &str) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1506` `let path = format!("/api/v1/admin/accounts/{}", account_id);`
- `sources/openviking/crates/ov_cli/src/client.rs:1507` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1508` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1509` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1510` `pub async fn admin_set_account_acl_enabled(`
- `sources/openviking/crates/ov_cli/src/client.rs:1511` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1573` `let path = format!("/api/v1/admin/accounts/{}/users/{}", account_id, user_id);`
- `sources/openviking/crates/ov_cli/src/client.rs:1574` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1575` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1576` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1577` `pub async fn admin_set_role(`
- `sources/openviking/crates/ov_cli/src/client.rs:1578` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1618` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1619` `pub async fn admin_delete_group(&self, account_id: &str, group_id: &str) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1620` `let path = format!("/api/v1/admin/accounts/{}/groups/{}", account_id, group_id);`
- `sources/openviking/crates/ov_cli/src/client.rs:1621` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1622` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1623` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1624` `pub async fn admin_list_group_members(`
- `sources/openviking/crates/ov_cli/src/client.rs:1625` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1658` `);`
- `sources/openviking/crates/ov_cli/src/client.rs:1659` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1660` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1661` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1662` `pub async fn admin_migrate(&self, cleanup: bool) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1663` `let action = if cleanup { "cleanup" } else { "migrate" };`
- `sources/openviking/crates/ov_cli/src/client.rs:1836` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1837` `pub async fn privacy_delete(`
- `sources/openviking/crates/ov_cli/src/client.rs:1838` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1839` `category: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1840` `target_key: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1841` `) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1842` `let path = format!("/api/v1/privacy-configs/{}/{}", category, target_key);`
- `sources/openviking/crates/ov_cli/src/client.rs:1843` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1844` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1845` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1846` `pub async fn privacy_delete_field(`
- `sources/openviking/crates/ov_cli/src/client.rs:1847` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1848` `category: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1849` `target_key: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1850` `field: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1852` `let path = format!("/api/v1/privacy-configs/{}/{}/field", category, target_key);`
- `sources/openviking/crates/ov_cli/src/client.rs:1853` `self.delete(&path, &[("field".to_string(), field.to_string())])`
- `sources/openviking/crates/ov_cli/src/client.rs:1854` `.await`
- `sources/openviking/crates/ov_cli/src/client.rs:1855` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1856` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1857` `pub async fn privacy_delete_selector(`
- `sources/openviking/crates/ov_cli/src/client.rs:1858` `&self,`
- `sources/openviking/crates/ov_cli/src/client.rs:1859` `selector: &str,`
- `sources/openviking/crates/ov_cli/src/client.rs:1860` `) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1861` `self.delete_with_body(`
- `sources/openviking/crates/ov_cli/src/client.rs:1862` `"/api/v1/privacy-configs/selector",`
- `sources/openviking/crates/ov_cli/src/client.rs:1863` `&serde_json::json!({"selector": selector}),`
- `sources/openviking/crates/ov_cli/src/client.rs:1864` `)`
- `sources/openviking/crates/ov_cli/src/client.rs:1865` `.await`
- `sources/openviking/crates/ov_cli/src/client.rs:1912` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1913` `pub async fn delete_watch_by_id(&self, task_id: &str) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1914` `let path = format!("/api/v1/watches/{}", task_id);`
- `sources/openviking/crates/ov_cli/src/client.rs:1915` `self.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1916` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1917` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1918` `pub async fn delete_watch_by_uri(&self, to_uri: &str) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1919` `let params = vec![("to_uri".to_string(), to_uri.to_string())];`
- `sources/openviking/crates/ov_cli/src/client.rs:1920` `self.delete("/api/v1/watches", &params).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1921` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1922` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1923` `pub async fn trigger_watch_by_id(&self, task_id: &str) -> Result<serde_json::Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1924` `let path = format!("/api/v1/watches/{}/trigger", task_id);`
- `sources/openviking/crates/ov_cli/src/client.rs:1991` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1992` `pub async fn snapshot_ignore_delete(&self) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/client.rs:1993` `self.delete("/api/v1/snapshot/ignore", &[]).await`
- `sources/openviking/crates/ov_cli/src/client.rs:1994` `}`
- `sources/openviking/crates/ov_cli/src/client.rs:1995` ``
- `sources/openviking/crates/ov_cli/src/client.rs:1996` `pub async fn snapshot_show(`
- `sources/openviking/crates/ov_cli/src/client.rs:1997` `&self,`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/acl.rs:84`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/acl.rs:83` `) -> Result<()> {`
- `sources/openviking/crates/ov_cli/src/commands/acl.rs:84` `show(client.acl_delete(uri).await?, output_format, compact)`
- `sources/openviking/crates/ov_cli/src/commands/acl.rs:85` `}`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/admin.rs:41`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/admin.rs:40` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:41` `pub async fn delete_account(`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:42` `client: &HttpClient,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:43` `account_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:44` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:45` `compact: bool,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:46` `) -> Result<()> {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:47` `let response = client.admin_delete_account(account_id).await?;`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:48` `let result =`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:49` `if response.is_null() || response.as_object().map(|o| o.is_empty()).unwrap_or(false) {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:50` `json!({"account_id": account_id})`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:51` `} else {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:212` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:213` `pub async fn delete_group(`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:214` `client: &HttpClient,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:215` `account_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:216` `group_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:217` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:220` `show_admin(`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:221` `client.admin_delete_group(account_id, group_id).await?,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:222` `output_format,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:223` `compact,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:224` `)`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:225` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:279` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:280` `invalidates_old: bool,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:281` `) {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:282` `let lines = admin_user_key_notice_lines(response, output_format, invalidates_old);`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:283` `if lines.is_empty() {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:284` `return;`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:285` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:286` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:295` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:296` `invalidates_old: bool,`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:297` `) -> Vec<String> {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:298` `if !matches!(output_format, OutputFormat::Table) || !contains_non_empty_user_key(response) {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:299` `return Vec::new();`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:300` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:325` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:326` `if invalidates_old {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:327` `lines.push(`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:328` `theme::warning("The old user key is invalidated immediately.")`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:329` `.bold()`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:330` `.to_string(),`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:331` `);`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:332` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:423` `assert!(rendered.contains("To use this key for normal commands, run ov config -> Edit config -> Set normal user API key."));`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:424` `assert!(!rendered.contains("old user key is invalidated"));`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:425` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:426` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:427` `#[test]`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:428` `fn user_key_notice_detects_nested_result_response() {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:456` `#[test]`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:457` `fn regenerate_key_notice_mentions_old_key_invalidation() {`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:458` `let response = json!({"user_key": "new-user-secret"});`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:459` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:460` `let rendered = admin_user_key_notice_lines(&response, OutputFormat::Table, true).join("\n");`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:461` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:462` `assert!(rendered.contains("The old user key is invalidated immediately."));`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:463` `}`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:464` ``
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:465` `#[test]`
- `sources/openviking/crates/ov_cli/src/commands/admin.rs:466` `fn list_users_table_output_redacts_plaintext_api_keys() {`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/crypto.rs:46`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:45` `"Key file already exists at: {}\n\`
- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:46` `Use a different path or delete the existing file first.",`
- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:47` `key_path.display()`
- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:48` `)));`
- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:49` `}`
- `sources/openviking/crates/ov_cli/src/commands/crypto.rs:50` ``

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:941`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:940` `let message = if let Some(count) = result`
- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:941` `.get("estimated_deleted_count")`
- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:942` `.and_then(|v| v.as_u64())`
- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:943` `{`
- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:944` `format!("Removed: {} ({} items)", uri, count)`
- `sources/openviking/crates/ov_cli/src/commands/filesystem.rs:945` `} else {`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/privacy.rs:388`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:387` ``
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:388` `pub async fn delete(`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:389` `client: &HttpClient,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:390` `selector_or_category: &str,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:391` `target_key: Option<&str>,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:392` `field: Option<&str>,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:402` `};`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:403` `if !yes && !confirm_delete(&display)? {`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:404` `output_success(`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:405` `&serde_json::json!({`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:406` `"selector": display,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:407` `"deleted": false,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:408` `"cancelled": true,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:409` `}),`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:410` `output_format,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:411` `compact,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:416` `let result = match (target_key, field) {`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:417` `(None, None) => client.privacy_delete_selector(selector_or_category).await?,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:418` `(Some(target), None) => client.privacy_delete(selector_or_category, target).await?,`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:419` `(Some(target), Some(field)) => {`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:420` `client`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:421` `.privacy_delete_field(selector_or_category, target, field)`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:422` `.await?`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:423` `}`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:424` `(None, Some(_)) => unreachable!("Clap cannot supply the third positional alone"),`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:425` `};`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:447` ``
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:448` `fn confirm_delete(display: &str) -> Result<bool> {`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:449` `if !io::stdin().is_terminal() || !io::stdout().is_terminal() {`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:450` `return Err(Error::Client(`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:451` `"Deleting a privacy target requires confirmation. Pass --yes to skip the prompt."`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:452` `.to_string(),`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:453` `));`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:454` `}`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:455` ``
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:456` `print!(`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:457` `"Delete Privacy selection {}? [y/N]: ",`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:458` `display`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:459` `);`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:460` `io::stdout().flush()?;`
- `sources/openviking/crates/ov_cli/src/commands/privacy.rs:461` `let mut answer = String::new();`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/session.rs:261`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/session.rs:260` ``
- `sources/openviking/crates/ov_cli/src/commands/session.rs:261` `pub async fn get_session_archive(`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:262` `client: &HttpClient,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:263` `session_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:264` `archive_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:265` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:266` `compact: bool,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:267` `) -> Result<()> {`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:268` `let path = format!(`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:269` `"/api/v1/sessions/{}/archives/{}",`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:270` `url_encode(session_id),`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:271` `url_encode(archive_id)`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:272` `);`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:273` `let response: serde_json::Value = client.get(&path, &[]).await?;`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:274` `output_success(&response, output_format, compact);`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:275` `Ok(())`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:277` ``
- `sources/openviking/crates/ov_cli/src/commands/session.rs:278` `pub async fn delete_session(`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:279` `client: &HttpClient,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:280` `session_id: &str,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:281` `output_format: OutputFormat,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:282` `compact: bool,`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:284` `let path = format!("/api/v1/sessions/{}", url_encode(session_id));`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:285` `let response: serde_json::Value = client.delete(&path, &[]).await?;`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:286` ``
- `sources/openviking/crates/ov_cli/src/commands/session.rs:287` `// Return session_id in result if empty (similar to Python implementation)`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:288` `let result =`
- `sources/openviking/crates/ov_cli/src/commands/session.rs:289` `if response.is_null() || response.as_object().map(|o| o.is_empty()).unwrap_or(false) {`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:99`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:98` `}`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:99` `SnapshotCmd::IgnoreDelete => {`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:100` `client.snapshot_ignore_delete().await?;`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:101` `output_success(&json!({ "result": "deleted" }), output_format, compact);`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:102` `Ok(())`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:103` `}`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:104` `}`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:105` `}`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:169` `"to_write": count("to_write"),`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:170` `"to_delete": count("to_delete"),`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:171` `"unchanged": count("unchanged"),`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:172` `})`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:173` `} else {`
- `sources/openviking/crates/ov_cli/src/commands/snapshot.rs:174` `value.clone()`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/commands/watch.rs:86`; body evidence.

- `sources/openviking/crates/ov_cli/src/commands/watch.rs:85` `let response = match classify_key(key)? {`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:86` `KeyKind::Uri => client.delete_watch_by_uri(key).await?,`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:87` `KeyKind::TaskId => client.delete_watch_by_id(key).await?,`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:88` `};`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:89` `output_success(&response, output_format, compact);`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:90` `Ok(())`
- `sources/openviking/crates/ov_cli/src/commands/watch.rs:91` `}`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/config_agent.rs:13`; body evidence.

- `sources/openviking/crates/ov_cli/src/config_agent.rs:12` `use crate::{`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:13` `CliContext, ConfigAddCustomArgs, ConfigAddOvServiceArgs, ConfigAddTarget, ConfigDeleteArgs,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:14` `ConfigEditArgs,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:15` `config::{Config, DEFAULT_CUSTOM_URL},`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:16` `config_wizard::{`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:17` `ApiKeyRole, ConfigKind, ConfigStore, OPENVIKING_SERVICE_URL, configs_equivalent,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:124` `#[derive(Debug, Clone, Serialize)]`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:125` `pub(crate) struct DeleteResult {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:126` `action: &'static str,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:127` `name: String,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:128` `deleted: bool,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:129` `already_absent: bool,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:130` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:131` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:132` `#[derive(Debug, Clone, Serialize)]`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:143` `Switch(SwitchResult),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:144` `Delete(DeleteResult),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:145` `List(Vec<ListEntry>),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:146` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:147` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:148` `pub(crate) async fn add(target: ConfigAddTarget, _ctx: &CliContext) -> AgentResult<AgentOutput> {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:194` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:195` `pub(crate) fn delete(args: ConfigDeleteArgs, _ctx: &CliContext) -> AgentResult<AgentOutput> {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:196` `let store = ConfigStore::new().map_err(config_error)?;`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:197` `delete_saved_config(&store, args)`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:198` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:199` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:200` `fn delete_saved_config(store: &ConfigStore, args: ConfigDeleteArgs) -> AgentResult<AgentOutput> {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:201` `validate_config_name(&args.name).map_err(config_error)?;`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:202` `let path = store.saved_config_path(&args.name).map_err(config_error)?;`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:203` `if !path.exists() {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:204` `return Ok(AgentOutput::Delete(DeleteResult {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:205` `action: "delete",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:206` `name: args.name,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:207` `deleted: false,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:208` `already_absent: true,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:209` `}));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:210` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:211` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:213` `return Err(AgentError::refused(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:214` `"Cannot delete the file selected by OPENVIKING_CLI_CONFIG_FILE. Change the environment variable first.",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:215` `));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:216` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:217` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:218` `match store.load_saved_config(&args.name) {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:224` `return Err(AgentError::refused(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:225` `"Cannot delete the active config. Run 'ov config switch <name>' first.",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:226` `));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:227` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:228` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:229` `Err(error) if !args.force => {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:230` `return Err(AgentError::bad_input(format!(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:231` `"Saved config '{}' cannot be read: {error}. Pass --force to delete the file.",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:232` `args.name`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:233` `)));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:234` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:235` `Err(_) => {}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:238` `fs::remove_file(&path).map_err(|error| {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:239` `AgentError::bad_input(format!("Failed to delete config '{}': {error}", args.name))`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:240` `})?;`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:241` `Ok(AgentOutput::Delete(DeleteResult {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:242` `action: "delete",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:243` `name: args.name,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:244` `deleted: true,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:245` `already_absent: false,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:246` `}))`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:247` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:248` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:254` `AgentOutput::Switch(result) => json!(result),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:255` `AgentOutput::Delete(result) => json!(result),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:256` `AgentOutput::List(result) => json!(result),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:257` `};`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:258` `println!("{}", json!({ "status": "ok", "result": result }));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:259` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:273` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:274` `AgentOutput::Delete(result) => {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:275` `if result.already_absent {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:276` `println!(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:277` `"{} {}",`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:278` `theme::success("✓").bold(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:284` `theme::success("✓").bold(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:285` `theme::success(format!("Deleted config '{}'.", result.name)).bold()`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:286` `);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:287` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:288` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:289` `AgentOutput::List(entries) => print_list_result(&entries),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1344` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1345` `fn deleting_missing_config_is_successful_noop() {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1346` `let dir = unique_dir("delete-missing");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1347` `fs::create_dir_all(&dir).expect("dir should exist");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1348` `let store = ConfigStore::for_config_dir(dir);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1349` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1350` `let output = delete_saved_config(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1351` `&store,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1352` `ConfigDeleteArgs {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1353` `name: "missing".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1354` `force: false,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1355` `},`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1356` `)`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1357` `.expect("missing delete should be idempotent");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1358` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1359` `let AgentOutput::Delete(result) = output else {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1360` `panic!("expected delete result");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1361` `};`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1362` `assert!(!result.deleted);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1363` `assert!(result.already_absent);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1364` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1365` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1366` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1367` `fn deleting_active_config_is_refused() {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1368` `let dir = unique_dir("delete-active");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1369` `fs::create_dir_all(&dir).expect("dir should exist");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1370` `let store = ConfigStore::for_config_dir(dir);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1371` `store`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1372` `.save_and_activate(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1377` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1378` `let error = delete_saved_config(`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1379` `&store,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1380` `ConfigDeleteArgs {`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1381` `name: "local".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1382` `force: true,`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1383` `},`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1384` `)`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1385` `.expect_err("active delete should be refused");`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1386` ``
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1387` `assert_eq!(error.exit_code(), EXIT_REFUSED);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1388` `}`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1389` `}`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/config_command_ui.rs:88`; body evidence.

- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:87` `language,`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:88` `"Add, edit, or delete configs",`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:89` `"添加、编辑或删除配置",`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:90` `),`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:91` `));`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:92` ``

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:303`; body evidence.

- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:302` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:303` `pub fn delete_config(&self, name: &str) -> Result<()> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:304` `if self.is_config_name_active(name)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:305` `return Err(Error::Config(`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:306` `"Cannot delete the active config. Switch to another config first.".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:307` `));`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:308` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:309` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:310` `let path = self.saved_config_path(name)?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:311` `fs::remove_file(&path)`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:312` `.map_err(|e| Error::Config(format!("Failed to delete config '{name}': {e}")))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:313` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:314` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:315` `pub(crate) fn selected_config_name(&self) -> Option<&str> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:316` `let name = self`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:881` `assert_eq!(store.load_active().unwrap().unwrap().url, "http://selected");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:882` `assert!(store.delete_config("z").is_err());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:883` `assert!(store.save_edited_config("z", "renamed", &config).is_err());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:884` `assert!(path.exists());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:885` `store.delete_config("a").unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:886` `store`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:887` `.save_active_config(&sample_config("http://edited", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:888` `.unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:889` `assert_eq!(`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1445` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1446` `fn delete_refuses_active_config() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1447` `let dir = unique_dir("delete-active");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1448` `let store = ConfigStore::for_config_dir(dir.clone());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1449` `store`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1450` `.save_named_config("local", &sample_config("http://local", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1451` `.expect("config should save");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1455` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1456` `assert!(store.delete_config("local").is_err());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1457` `assert!(dir.join("ovcli.conf.local").exists());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1458` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1459` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1460` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1461` `fn delete_duplicate_non_active_config_is_allowed() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1462` `let dir = unique_dir("delete-duplicate");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1463` `let store = ConfigStore::for_config_dir(dir.clone());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1464` `let config = sample_config("http://local", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1465` `store`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1466` `.save_named_config("local", &config)`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1475` `store`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1476` `.delete_config("local-copy")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1477` `.expect("non-active duplicate should delete");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1478` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1479` `assert!(dir.join("ovcli.conf.local").exists());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1480` `assert!(!dir.join("ovcli.conf.local-copy").exists());`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1481` `assert!(dir.join("ovcli.conf").exists());`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:135`; body evidence.

- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:134` `PromptResult::Value(3) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:135` `if run_delete_config(&store, &mut ui)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:136` `return Ok(());`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:137` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:138` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:139` `PromptResult::Value(4) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:322` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:323` `fn section_delete() -> &'static str {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:324` `copy(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:325` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:326` `"Delete a saved config.",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:327` `"删除已保存的配置。",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:328` `)`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:329` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:330` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:541` `"Edit Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:542` `"Delete Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:543` `"User Management",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:544` `]`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:545` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:546` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2887` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2888` `fn run_delete_config(store: &ConfigStore, ui: &mut LiveRegion) -> Result<bool> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2889` `enum Stage {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2890` `Select,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2891` `Confirm,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2892` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2898` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2899` `"No saved configs to delete.",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2900` `"没有可删除的配置。",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2901` `))`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2902` `.to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2903` `];`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2905` `ui,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2906` `section_delete(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2907` `copy(Language::current(), "Nothing to delete.", "没有可删除项。"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2908` `&[copy(Language::current(), "Back", "返回")],`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2909` `0,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2910` `&helper_lines,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2911` `)?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2921` `ui,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2922` `section_delete(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2923` `copy(Language::current(), "Config to delete", "要删除的配置"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2924` `&configs,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2925` `)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2926` `PromptResult::Value(index) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2927` `selected = index;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2928` `if configs[index].is_active {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2929` `let helper_lines = active_delete_block_helper_lines();`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2930` `let _ = prompt_select(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2931` `ui,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2932` `section_delete(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2933` `copy(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2934` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2935` `"Active config cannot be deleted.",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2936` `"不能删除当前配置。",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2937` `),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2938` `&[copy(Language::current(), "Back", "返回")],`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2939` `0,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2953` `let name = &configs[selected].name;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2954` `match confirm(ui, section_delete(), &delete_confirm_prompt(name), false)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2955` `PromptResult::Value(true) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2956` `ui.clear()?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2957` `store.delete_config(name)?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2958` `println!();`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2959` `println!(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2960` `"{} {}",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2961` `theme::success("✓"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2962` `theme::success(deleted_config_message(name))`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2963` `);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2964` `println!(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2965` `"{} {}",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:2966` `theme::muted(copy(Language::current(), "Removed:", "已删除：")),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3507` `labels.push(add_user_label(&account.account_id));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3508` `labels.push(delete_account_label(&account.account_id));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3509` `labels.push(back_label());`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3510` `let helper_lines = account_management_menu_helper_lines(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3511` `config_name.as_deref(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3512` `config,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3552` `let add_user_index = users.len();`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3553` `let delete_account_index = users.len() + 1;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3554` `let back_index = users.len() + 2;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3555` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3556` `if selected == add_user_index {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3557` `match create_user_in_account(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3579` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3580` `if selected == delete_account_index {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3581` `match confirm(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3582` `ui,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3583` `section_user_management(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3584` `&delete_account_prompt(&account.account_id),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3585` `false,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3586` `)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3587` `PromptResult::Value(true) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3588` `ui.render(&status_live_lines(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3591` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3592` `"Deleting account...",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3593` `"正在删除账户...",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3594` `),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3595` `))?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3596` `let task = delete_root_account(&client, &account.account_id).await?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3597` `if clear_config_account_selection(config, root_api_key, &account.account_id) {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3598` `save_user_management_local_config(store, config_name.as_deref(), config)?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3599` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3600` `let task_id = task["task_id"].as_str().ok_or_else(|| {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3601` `Error::Parse("Account deletion response has no task_id".to_string())`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3602` `})?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3603` `*account_list_notice =`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3604` `Some(account_deletion_notice(&account.account_id, task_id));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3605` `return Ok(false);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3606` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3607` `PromptResult::Value(false) | PromptResult::Back => continue,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3608` `PromptResult::Quit => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3634` `RegenerateAndUse,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3635` `Delete,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3636` `Back,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3637` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3638` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3639` `loop {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3647` `labels.push(regenerate_and_use_user_label());`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3648` `actions.push(UserAction::Delete);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3649` `labels.push(delete_user_label(account_id, &user.user_id));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3650` `actions.push(UserAction::Back);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3651` `labels.push(back_label());`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3652` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3653` `let helper_lines =`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3701` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3702` `UserAction::Delete => match confirm(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3703` `ui,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3704` `section_user_management(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3705` `&delete_user_prompt(account_id, &user.user_id),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3706` `false,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3707` `)? {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3708` `PromptResult::Value(true) => {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3709` `let client = root_admin_client(config, root_api_key);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3711` `section_user_management(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3712` `copy(Language::current(), "Deleting user...", "正在删除用户..."),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3713` `))?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3714` `delete_root_user(&client, account_id, &user.user_id).await?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3715` `if clear_config_user_selection(config, root_api_key, account_id, &user.user_id)`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3716` `{`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3717` `save_user_management_local_config(store, config_name.as_deref(), config)?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3718` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3910` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3911` `"Regenerating a user key invalidates the old key immediately.",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3912` `"重新生成用户 Key 会立即使旧 Key 失效。",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3913` `))`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3914` `.to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:3915` `);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4016` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4017` `fn account_deletion_notice(account_id: &str, task_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4018` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4019` `Language::En => format!("Account {account_id}/ disabled. Cleanup task: {task_id}"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4020` `Language::ZhCn => format!("账户 {account_id}/ 已停用，后台清理任务：{task_id}"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4021` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4023` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4024` `fn deleted_user_notice(account_id: &str, user_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4025` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4026` `Language::En => format!("Deleted user {account_id}/{user_id}."),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4027` `Language::ZhCn => format!("已删除用户 {account_id}/{user_id}。"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4028` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4029` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4030` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4150` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4151` `fn delete_account_label(account_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4152` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4153` `Language::En => format!("- Delete {account_id}/"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4154` `Language::ZhCn => format!("- 删除 {account_id}/"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4155` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4156` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4157` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4158` `fn delete_user_label(account_id: &str, user_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4159` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4160` `Language::En => format!("- Delete {account_id}/{user_id}"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4161` `Language::ZhCn => format!("- 删除 {account_id}/{user_id}"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4162` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4163` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4164` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4202` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4203` `fn delete_account_prompt(account_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4204` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4205` `Language::En => format!("Delete account {account_id}/ and all of its users?"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4206` `Language::ZhCn => format!("删除账户 {account_id}/ 及其所有用户？"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4207` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4208` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4209` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4210` `fn delete_user_prompt(account_id: &str, user_id: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4211` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4212` `Language::En => format!("Delete user {account_id}/{user_id}?"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4213` `Language::ZhCn => format!("删除用户 {account_id}/{user_id}？"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4214` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4215` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4216` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4568` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4569` `async fn delete_root_account(client: &BaseClient, account_id: &str) -> Result<Value> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4570` `let path = format!("/api/v1/admin/accounts/{account_id}");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4571` `client.delete(&path, &[]).await`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4572` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4573` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4574` `async fn delete_root_user(client: &BaseClient, account_id: &str, user_id: &str) -> Result<()> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4575` `let path = format!("/api/v1/admin/accounts/{account_id}/users/{user_id}");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4576` `let _: Value = client.delete(&path, &[]).await?;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4577` `Ok(())`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4578` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4579` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4580` `async fn regenerate_user_key(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4596` `if account_id.is_empty()`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4597` `|| item.get("status").and_then(Value::as_str) == Some("deleting")`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4598` `{`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4599` `return None;`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4600` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4601` `Some(RootAccountSummary {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4790` `Language::current(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4791` `"This server does not expose the existing user key. Regenerating invalidates the old key immediately.",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4792` `"服务端未返回现有用户 Key。重新生成会立即使旧 Key 失效。",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4793` `))`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4794` `.to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:4795` `]`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5383` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5384` `fn active_delete_block_helper_lines() -> Vec<String> {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5385` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5386` `Language::En => vec![`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5387` `theme::error("Deleting the active config is blocked.").to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5388` `format!(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5389` `"{} {} {}",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5390` `theme::muted("Run"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5391` `theme::command("ov config switch").bold(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5392` `theme::muted("to choose another config, then delete this one.")`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5393` `),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5394` `],`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5395` `Language::ZhCn => vec![`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5396` `theme::error("不能删除当前配置。").to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5406` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5407` `fn delete_confirm_prompt(name: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5408` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5409` `Language::En => format!("Delete config '{name}'?"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5410` `Language::ZhCn => format!("删除配置 '{name}'？"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5411` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5412` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5413` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5427` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5428` `fn deleted_config_message(name: &str) -> String {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5429` `match Language::current() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5430` `Language::En => format!("Deleted config '{name}'."),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5431` `Language::ZhCn => format!("已删除配置 '{name}'。"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5432` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5433` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:5434` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6122` `RootUserSummary, StatusBoxFrame, StatusBoxMode, StatusBoxRuntime, account_tree_labels,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6123` `active_config_index, active_delete_block_helper_lines, active_summary_lines,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6124` `active_summary_render_parts, add_config_name_label, add_generated_config_name_helper_lines,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6125` `add_save_action_labels, allocate_config_name, box_content_line, box_footer_line,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6126` `box_title_line, compact_status_box_width, config_select_label, config_with_selected_user,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6127` `current_server_url_default, custom_api_key_helper_lines, custom_api_key_input_helper_lines,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7453` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7454` `fn active_delete_copy_mentions_switch_command() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7455` `let copy = active_delete_block_helper_lines().join("\n");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7456` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7457` `assert!(copy.contains("Deleting the active config is blocked."));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7458` `assert!(copy.contains("ov config switch"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7459` `assert!(copy.contains("then delete this one"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7460` `}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7461` ``
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7462` `#[test]`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7463` `fn save_choices_allow_saving_without_activation() {`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7961` `"Edit Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7962` `"Delete Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7963` `],`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7964` `1,`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7965` `&[],`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7966` `);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7982` `"Edit Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7983` `"Delete Config",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7984` `"User Management",`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7985` `]`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7986` `.as_slice()`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7987` `);`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8060` `{"account_id": "", "user_count": 9},`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8061` `{"account_id": "deleting-account", "user_count": 2, "status": "deleting"},`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8062` `{"account_id": "alpha", "user_count": 2}`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8063` `]));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8064` `assert_eq!(`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8065` `accounts,`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/error_ui.rs:132`; body evidence.

- `sources/openviking/crates/ov_cli/src/error_ui.rs:131` `language,`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:132` `"Use ov config to add, edit, or delete configs.",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:133` `"请使用 ov config 添加、编辑或删除配置。",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:134` `),`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:135` `)`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:136` `.with_command(command)`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:143` `language,`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:144` `"Add, edit, or delete configs",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:145` `"添加、编辑或删除配置",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:146` `),`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:147` `),`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:148` `ErrorAction::new(`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:363` `}`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:364` `Error::Zip(error) => ErrorReport::new(copy(language, "Archive Error", "压缩包错误"), copy(language, "OpenViking could not process the archive.", "OpenViking 无法处理压缩包。"))`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:365` `.with_command(command)`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:366` `.with_detail(error.to_string()),`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:367` `Error::AlreadyReported => ErrorReport::new(copy(language, "Command Error", "命令错误"), copy(language, "The command failed.", "命令执行失败。"))`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:368` `.with_command(command),`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:430` `"list-accounts",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:431` `"delete-account",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:432` `"migrate",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:433` `"register-user",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:434` `"list-users",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:435` `"create-group",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:439` `"remove-group-member",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:440` `"delete-group",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:441` `"remove-user",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:442` `"set-role",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:443` `"regenerate-key",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:444` `"set-account-settings",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:456` `"get-session-context",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:457` `"get-session-archive",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:458` `"delete",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:459` `"add-message",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:460` `"add-messages",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:461` `"commit",`
- `sources/openviking/crates/ov_cli/src/error_ui.rs:462` `],`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/handlers.rs:507`; body evidence.

- `sources/openviking/crates/ov_cli/src/handlers.rs:506` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:507` `SessionCommands::GetSessionArchive {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:508` `session_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:509` `archive_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:510` `} => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:511` `commands::session::get_session_archive(`
- `sources/openviking/crates/ov_cli/src/handlers.rs:512` `&client,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:513` `&session_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:514` `&archive_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:515` `ctx.output_format,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:516` `ctx.compact,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:517` `)`
- `sources/openviking/crates/ov_cli/src/handlers.rs:518` `.await`
- `sources/openviking/crates/ov_cli/src/handlers.rs:519` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:520` `SessionCommands::Delete { session_id } => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:521` `commands::session::delete_session(&client, &session_id, ctx.output_format, ctx.compact)`
- `sources/openviking/crates/ov_cli/src/handlers.rs:522` `.await`
- `sources/openviking/crates/ov_cli/src/handlers.rs:523` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:524` `SessionCommands::AddMessage {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:525` `session_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:618` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:619` `AdminCommands::DeleteAccount { account_id } => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:620` `commands::admin::delete_account(&client, &account_id, ctx.output_format, ctx.compact)`
- `sources/openviking/crates/ov_cli/src/handlers.rs:621` `.await`
- `sources/openviking/crates/ov_cli/src/handlers.rs:622` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:623` `AdminCommands::Migrate { cleanup } => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:624` `commands::admin::migrate(&client, cleanup, ctx.output_format, ctx.compact).await`
- `sources/openviking/crates/ov_cli/src/handlers.rs:722` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:723` `AdminCommands::DeleteGroup {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:724` `account_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:725` `group_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:726` `} => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:727` `commands::admin::delete_group(`
- `sources/openviking/crates/ov_cli/src/handlers.rs:728` `&client,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:729` `&account_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:730` `&group_id,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:731` `ctx.output_format,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:918` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:919` `PrivacyCommands::Delete {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:920` `selector_or_category,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:921` `target_key,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:922` `field,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:923` `yes,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:924` `} => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:925` `commands::privacy::delete(`
- `sources/openviking/crates/ov_cli/src/handlers.rs:926` `&client,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:927` `&selector_or_category,`
- `sources/openviking/crates/ov_cli/src/handlers.rs:928` `target_key.as_deref(),`
- `sources/openviking/crates/ov_cli/src/handlers.rs:929` `field.as_deref(),`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1003` `Some(ConfigCommands::List) => handle_config_agent_result(config_agent::list(&ctx), &ctx),`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1004` `Some(ConfigCommands::Delete(args)) => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1005` `handle_config_agent_result(config_agent::delete(args, &ctx), &ctx)`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1006` `}`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1007` `Some(ConfigCommands::Add { target }) => {`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1008` `let result = config_agent::add(target, &ctx).await;`
- `sources/openviking/crates/ov_cli/src/handlers.rs:1009` `handle_config_agent_result(result, &ctx)`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/help_ui.rs:387`; body evidence.

- `sources/openviking/crates/ov_cli/src/help_ui.rs:386` `label: "ov acl set viking://resources/project-a --acl-mode restricted",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:387` `description: "Ignore inherited permissions without deleting them.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:388` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:389` `],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:390` `next_steps: &[HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:391` `label: "ov find \"query\" -u <uri>",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:568` `path: &["session"],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:569` `purpose: "Manage sessions, messages, archives, and committed session context.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:570` `examples: &[`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:571` `HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:572` `label: "ov session new",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:573` `description: "Create a new session.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:743` `CommandHelpSpec {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:744` `path: &["snapshot", "ignore-delete"],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:745` `purpose: "Delete the account-level .ovgitignore file (idempotent).",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:746` `examples: &[HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:747` `label: "ov snapshot ignore-delete",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:748` `description: "Remove the ignore rules; missing is success.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:749` `}],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:750` `next_steps: &[HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:751` `label: "ov snapshot ignore-get",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1054` `path: &["config"],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1055` `purpose: "Add, edit, delete, show, validate, or switch OpenViking CLI configs.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1056` `examples: &[`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1057` `HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1058` `label: "ov config",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1059` `description: "Open the interactive config manager.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1269` `CommandHelpSpec {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1270` `path: &["config", "delete"],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1271` `purpose: "Delete a saved CLI config without prompts.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1272` `examples: &[`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1273` `HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1274` `label: "ov config delete old-local",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1275` `description: "Delete a non-active saved config.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1276` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1277` `HelpItem {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1278` `label: "ov config delete missing -o json",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1279` `description: "Return a JSON no-op if the config is already absent.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1280` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1281` `],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1282` `next_steps: &[`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1288` `label: "ov config switch <name>",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1289` `description: "Switch away from an active config before deleting it.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1290` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1291` `],`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1292` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1293` `CommandHelpSpec {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1930` `["config", "edit"] => "不打开交互式向导，编辑已保存的 CLI 配置。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1931` `["config", "delete"] => "不打开交互式向导，删除已保存的 CLI 配置。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1932` `["health"] => "快速检查服务器是否可连接。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1933` `["status"] => "查看 OpenViking 服务器诊断状态。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1934` `["language"] => "选择 OpenViking CLI 显示语言。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1935` `["snapshot"] => "管理工作区快照：提交、恢复、查看、对比，以及遍历历史。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1961` `"edit" => "不打开提示，编辑已保存配置。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1962` `"delete" => "不打开提示，删除已保存配置。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1963` `"ov-service" => "使用固定的 OpenViking 服务（火山引擎云）地址。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1964` `"custom" => "使用本地或远程自定义地址。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1965` `"ov --help" => "查看所有命令。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:1966` `"ov health" => "快速健康检查。",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2389` `"config list" => "列出已保存配置",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2390` `"config delete" => "删除已保存配置",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2391` `"health" => "快速检查服务器连接",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2392` `"status" => "查看系统状态",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2393` `"wait" => "等待异步任务完成",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2394` `"task" => "查看异步任务",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2530` `[base] if base == "config" => match token.as_str() {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2531` `"show" | "validate" | "switch" | "list" | "delete" | "edit" | "add" => {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2532` `path.push(token.clone());`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2533` `}`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2534` `_ => return Some(path),`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2535` `},`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2598` `"list" => "ls",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2599` `"del" | "delete" => "rm",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2600` `"rename" => "mv",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2601` `"lang" => "language",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2602` `other => other,`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2603` `}`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2798` `"config list",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2799` `"config delete",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2800` `] {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2801` `assert!(`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2802` `!rendered.contains(nested),`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:2803` `"top-level help leaked nested command {nested}"`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3110` ``
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3111` `let delete = strip_ansi(`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3112` `&render_command_help_request(&os_args(&["ov", "config", "delete", "prod", "--help"]))`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3113` `.expect("config delete help should render"),`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3114` `);`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3115` `assert!(delete.contains("ov config delete [OPTIONS] <name>"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3116` `assert!(delete.contains("Delete even when the saved config file cannot be parsed"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3117` `}`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3118` ``
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3119` `#[test]`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3120` `fn renders_curated_group_help_for_config_and_task() {`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3130` `assert!(config.contains("list"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3131` `assert!(config.contains("delete"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3132` ``
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3133` `let task = strip_ansi(`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3134` `&render_command_help_request(&os_args(&["ov", "task", "--help"]))`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3135` `.expect("task help should render"),`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3150` `assert!(session.contains("get-session-context <session-id>"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3151` `assert!(session.contains("get-session-archive <session-id> <archive-id>"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3152` `assert!(session.contains("add-message <session-id>"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3153` `assert!(session.contains("add-messages <session-id> <messages-json>"));`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3154` ``
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3155` `let watch = strip_ansi(`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3187` `"session",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3188` `"Manage sessions, messages, archives, and committed session context.",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3189` `),`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3190` `(`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3191` `"privacy",`
- `sources/openviking/crates/ov_cli/src/help_ui.rs:3192` `"Manage privacy targets, field references, values, versions, and cleanup.",`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/main.rs:717`; body evidence.

- `sources/openviking/crates/ov_cli/src/main.rs:716` `/// [Data] Remove resource`
- `sources/openviking/crates/ov_cli/src/main.rs:717` `#[command(alias = "del", alias = "delete")]`
- `sources/openviking/crates/ov_cli/src/main.rs:718` `Rm {`
- `sources/openviking/crates/ov_cli/src/main.rs:719` `/// Viking URI to remove`
- `sources/openviking/crates/ov_cli/src/main.rs:720` `#[arg(value_name = "uri")]`
- `sources/openviking/crates/ov_cli/src/main.rs:721` `uri: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:1541` `/// Delete the account .ovgitignore`
- `sources/openviking/crates/ov_cli/src/main.rs:1542` `IgnoreDelete,`
- `sources/openviking/crates/ov_cli/src/main.rs:1543` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:1544` ``
- `sources/openviking/crates/ov_cli/src/main.rs:1545` `#[derive(Subcommand)]`
- `sources/openviking/crates/ov_cli/src/main.rs:1546` `enum SystemCommands {`
- `sources/openviking/crates/ov_cli/src/main.rs:1621` `/// Get one completed archive for a session`
- `sources/openviking/crates/ov_cli/src/main.rs:1622` `GetSessionArchive {`
- `sources/openviking/crates/ov_cli/src/main.rs:1623` `/// Session ID`
- `sources/openviking/crates/ov_cli/src/main.rs:1624` `#[arg(value_name = "session-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:1625` `session_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:1626` `/// Archive ID`
- `sources/openviking/crates/ov_cli/src/main.rs:1627` `#[arg(value_name = "archive-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:1628` `archive_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:1629` `},`
- `sources/openviking/crates/ov_cli/src/main.rs:1630` `/// Delete a session`
- `sources/openviking/crates/ov_cli/src/main.rs:1631` `Delete {`
- `sources/openviking/crates/ov_cli/src/main.rs:1632` `/// Session ID`
- `sources/openviking/crates/ov_cli/src/main.rs:1633` `#[arg(value_name = "session-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:1634` `session_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:1635` `},`
- `sources/openviking/crates/ov_cli/src/main.rs:1856` `/// Remove installed skills`
- `sources/openviking/crates/ov_cli/src/main.rs:1857` `#[command(alias = "rm", alias = "delete")]`
- `sources/openviking/crates/ov_cli/src/main.rs:1858` `Remove {`
- `sources/openviking/crates/ov_cli/src/main.rs:1859` `/// Skill name(s) to remove`
- `sources/openviking/crates/ov_cli/src/main.rs:1860` `#[arg(value_name = "skill")]`
- `sources/openviking/crates/ov_cli/src/main.rs:1861` `skills: Vec<String>,`
- `sources/openviking/crates/ov_cli/src/main.rs:2046` `/// Delete a target or make one active field unavailable`
- `sources/openviking/crates/ov_cli/src/main.rs:2047` `Delete {`
- `sources/openviking/crates/ov_cli/src/main.rs:2048` `/// Privacy reference, URI, or config category`
- `sources/openviking/crates/ov_cli/src/main.rs:2049` `#[arg(value_name = "reference-or-uri-or-category")]`
- `sources/openviking/crates/ov_cli/src/main.rs:2050` `selector_or_category: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:2051` `/// Privacy config target key (category form only)`
- `sources/openviking/crates/ov_cli/src/main.rs:2108` `/// Delete an account and all associated users (ROOT only)`
- `sources/openviking/crates/ov_cli/src/main.rs:2109` `DeleteAccount {`
- `sources/openviking/crates/ov_cli/src/main.rs:2110` `/// Account ID to delete`
- `sources/openviking/crates/ov_cli/src/main.rs:2111` `#[arg(value_name = "account-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:2112` `account_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:2113` `},`
- `sources/openviking/crates/ov_cli/src/main.rs:2193` `/// Delete an empty group`
- `sources/openviking/crates/ov_cli/src/main.rs:2194` `DeleteGroup {`
- `sources/openviking/crates/ov_cli/src/main.rs:2195` `#[arg(value_name = "account-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:2196` `account_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:2197` `#[arg(value_name = "group-id")]`
- `sources/openviking/crates/ov_cli/src/main.rs:2198` `group_id: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:2258` `| ConfigCommands::Edit(_)`
- `sources/openviking/crates/ov_cli/src/main.rs:2259` `| ConfigCommands::Delete(_)`
- `sources/openviking/crates/ov_cli/src/main.rs:2260` `| ConfigCommands::List,`
- `sources/openviking/crates/ov_cli/src/main.rs:2261` `),`
- `sources/openviking/crates/ov_cli/src/main.rs:2262` `} | Commands::Skills {`
- `sources/openviking/crates/ov_cli/src/main.rs:2263` `action: SkillCommands::Validate { .. },`
- `sources/openviking/crates/ov_cli/src/main.rs:2287` `/// Delete a saved config`
- `sources/openviking/crates/ov_cli/src/main.rs:2288` `Delete(ConfigDeleteArgs),`
- `sources/openviking/crates/ov_cli/src/main.rs:2289` `/// Add a saved config without opening the interactive wizard`
- `sources/openviking/crates/ov_cli/src/main.rs:2290` `Add {`
- `sources/openviking/crates/ov_cli/src/main.rs:2291` `#[command(subcommand)]`
- `sources/openviking/crates/ov_cli/src/main.rs:2292` `target: ConfigAddTarget,`
- `sources/openviking/crates/ov_cli/src/main.rs:2477` `#[derive(Args, Debug, Clone)]`
- `sources/openviking/crates/ov_cli/src/main.rs:2478` `struct ConfigDeleteArgs {`
- `sources/openviking/crates/ov_cli/src/main.rs:2479` `/// Saved config name to delete`
- `sources/openviking/crates/ov_cli/src/main.rs:2480` `#[arg(value_name = "name")]`
- `sources/openviking/crates/ov_cli/src/main.rs:2481` `name: String,`
- `sources/openviking/crates/ov_cli/src/main.rs:2482` `/// Delete even when the saved config file cannot be parsed`
- `sources/openviking/crates/ov_cli/src/main.rs:2597` `"list" => "ls",`
- `sources/openviking/crates/ov_cli/src/main.rs:2598` `"del" | "delete" => "rm",`
- `sources/openviking/crates/ov_cli/src/main.rs:2599` `"rename" => "mv",`
- `sources/openviking/crates/ov_cli/src/main.rs:2600` `"lang" => "language",`
- `sources/openviking/crates/ov_cli/src/main.rs:2601` `other => other,`
- `sources/openviking/crates/ov_cli/src/main.rs:2602` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:2675` `match tokens.get(1).map(String::as_str) {`
- `sources/openviking/crates/ov_cli/src/main.rs:2676` `Some("add" | "edit" | "delete" | "list") => true,`
- `sources/openviking/crates/ov_cli/src/main.rs:2677` `Some("switch") => tokens.get(2).is_some(),`
- `sources/openviking/crates/ov_cli/src/main.rs:2678` `_ => false,`
- `sources/openviking/crates/ov_cli/src/main.rs:2679` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:2680` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:2810` `token,`
- `sources/openviking/crates/ov_cli/src/main.rs:2811` `"add" | "list" | "ls" | "find" | "show" | "update" | "remove" | "rm" | "delete"`
- `sources/openviking/crates/ov_cli/src/main.rs:2812` `)`
- `sources/openviking/crates/ov_cli/src/main.rs:2813` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:2814` ``
- `sources/openviking/crates/ov_cli/src/main.rs:2815` `fn is_admin_subcommand(token: &str) -> bool {`
- `sources/openviking/crates/ov_cli/src/main.rs:2819` `| "list-accounts"`
- `sources/openviking/crates/ov_cli/src/main.rs:2820` `| "delete-account"`
- `sources/openviking/crates/ov_cli/src/main.rs:2821` `| "migrate"`
- `sources/openviking/crates/ov_cli/src/main.rs:2822` `| "register-user"`
- `sources/openviking/crates/ov_cli/src/main.rs:2823` `| "list-users"`
- `sources/openviking/crates/ov_cli/src/main.rs:2824` `| "create-group"`
- `sources/openviking/crates/ov_cli/src/main.rs:2828` `| "remove-group-member"`
- `sources/openviking/crates/ov_cli/src/main.rs:2829` `| "delete-group"`
- `sources/openviking/crates/ov_cli/src/main.rs:2830` `| "remove-user"`
- `sources/openviking/crates/ov_cli/src/main.rs:2831` `| "set-role"`
- `sources/openviking/crates/ov_cli/src/main.rs:2832` `| "regenerate-key"`
- `sources/openviking/crates/ov_cli/src/main.rs:2833` `| "set-account-settings"`
- `sources/openviking/crates/ov_cli/src/main.rs:2850` `| "get-session-context"`
- `sources/openviking/crates/ov_cli/src/main.rs:2851` `| "get-session-archive"`
- `sources/openviking/crates/ov_cli/src/main.rs:2852` `| "delete"`
- `sources/openviking/crates/ov_cli/src/main.rs:2853` `| "add-message"`
- `sources/openviking/crates/ov_cli/src/main.rs:2854` `| "add-messages"`
- `sources/openviking/crates/ov_cli/src/main.rs:2855` `| "commit"`
- `sources/openviking/crates/ov_cli/src/main.rs:2856` `)`
- `sources/openviking/crates/ov_cli/src/main.rs:2870` `| "activate"`
- `sources/openviking/crates/ov_cli/src/main.rs:2871` `| "delete"`
- `sources/openviking/crates/ov_cli/src/main.rs:2872` `| "scan"`
- `sources/openviking/crates/ov_cli/src/main.rs:2873` `)`
- `sources/openviking/crates/ov_cli/src/main.rs:2874` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:2875` ``
- `sources/openviking/crates/ov_cli/src/main.rs:4690` `&["ov", "admin", "list-accounts"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4691` `&["ov", "admin", "delete-account"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4692` `&["ov", "admin", "migrate"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4693` `&["ov", "admin", "register-user"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4694` `&["ov", "admin", "list-users"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4695` `&["ov", "admin", "create-group"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4699` `&["ov", "admin", "remove-group-member"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4700` `&["ov", "admin", "delete-group"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4701` `&["ov", "admin", "remove-user"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4702` `&["ov", "admin", "set-role"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4703` `&["ov", "admin", "regenerate-key"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4704` `&["ov", "admin", "set-account-settings"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4714` `&["ov", "session", "get-session-context"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4715` `&["ov", "session", "get-session-archive"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4716` `&["ov", "session", "delete"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4717` `&["ov", "session", "add-message"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4718` `&["ov", "session", "add-messages"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4719` `&["ov", "session", "commit"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4720` `&["ov", "privacy", "categories"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4728` `&["ov", "privacy", "activate"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4729` `&["ov", "privacy", "delete"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4730` `&["ov", "privacy", "scan"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4731` `&["ov", "observer", "queue"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4732` `&["ov", "observer", "vikingdb"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4733` `&["ov", "observer", "models"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4744` `&["ov", "skills", "rm"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4745` `&["ov", "skills", "delete"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4746` `];`
- `sources/openviking/crates/ov_cli/src/main.rs:4747` ``
- `sources/openviking/crates/ov_cli/src/main.rs:4748` `for args in cases {`
- `sources/openviking/crates/ov_cli/src/main.rs:4749` `assert!(`
- `sources/openviking/crates/ov_cli/src/main.rs:4766` `&["ov", "list"][..],`
- `sources/openviking/crates/ov_cli/src/main.rs:4767` `&["ov", "delete"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4768` `&["ov", "rename"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4769` `&["ov", "lang", "en"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4770` `] {`
- `sources/openviking/crates/ov_cli/src/main.rs:4771` `let expected = args[1] != "lang";`
- `sources/openviking/crates/ov_cli/src/main.rs:4788` `&["ov", "config", "list"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4789` `&["ov", "config", "delete", "prod"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4790` `&["ov", "config", "add", "ov-service", "--api-key-stdin"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4791` `&["ov", "config", "edit", "prod", "--activate"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4792` `&["ov", "config", "setup-cli"],`
- `sources/openviking/crates/ov_cli/src/main.rs:4793` `&["ov", "skills", "validate", "./skills/foo"],`
- `sources/openviking/crates/ov_cli/src/main.rs:5485` `&["ov", "config", "edit", "prod", "--activate"],`
- `sources/openviking/crates/ov_cli/src/main.rs:5486` `&["ov", "config", "delete", "prod"],`
- `sources/openviking/crates/ov_cli/src/main.rs:5487` `&["ov", "config", "list"],`
- `sources/openviking/crates/ov_cli/src/main.rs:5488` `&["ov", "config", "switch", "prod"],`
- `sources/openviking/crates/ov_cli/src/main.rs:5489` `] {`
- `sources/openviking/crates/ov_cli/src/main.rs:5490` `assert_eq!(`
- `sources/openviking/crates/ov_cli/src/main.rs:6110` `#[test]`
- `sources/openviking/crates/ov_cli/src/main.rs:6111` `fn cli_parses_privacy_reference_resolve_and_delete() {`
- `sources/openviking/crates/ov_cli/src/main.rs:6112` `on_cli_parser_stack(|| {`
- `sources/openviking/crates/ov_cli/src/main.rs:6113` `for (subcommand, expected_field) in [("reference", "api_key"), ("resolve", "token")] {`
- `sources/openviking/crates/ov_cli/src/main.rs:6114` `let cli = Cli::parse_from(preprocess_privacy_args(vec![`
- `sources/openviking/crates/ov_cli/src/main.rs:6115` `OsString::from("ov"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6243` `OsString::from("privacy"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6244` `OsString::from("delete"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6245` `OsString::from("credential"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6246` `OsString::from("github-prod"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6247` `OsString::from("--yes"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6248` `]));`
- `sources/openviking/crates/ov_cli/src/main.rs:6250` `Commands::Privacy { action } => match *action {`
- `sources/openviking/crates/ov_cli/src/main.rs:6251` `PrivacyCommands::Delete {`
- `sources/openviking/crates/ov_cli/src/main.rs:6252` `selector_or_category,`
- `sources/openviking/crates/ov_cli/src/main.rs:6253` `target_key,`
- `sources/openviking/crates/ov_cli/src/main.rs:6254` `field,`
- `sources/openviking/crates/ov_cli/src/main.rs:6255` `yes,`
- `sources/openviking/crates/ov_cli/src/main.rs:6261` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:6262` `_ => panic!("expected privacy delete"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6263` `},`
- `sources/openviking/crates/ov_cli/src/main.rs:6264` `_ => panic!("expected privacy command"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6265` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:6266` ``
- `sources/openviking/crates/ov_cli/src/main.rs:6267` `let field_delete = Cli::parse_from(preprocess_privacy_args(vec![`
- `sources/openviking/crates/ov_cli/src/main.rs:6268` `OsString::from("ov"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6269` `OsString::from("privacy"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6270` `OsString::from("delete"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6271` `OsString::from("credential"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6272` `OsString::from("github-prod"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6273` `OsString::from("api_key"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6274` `OsString::from("--yes"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6275` `]));`
- `sources/openviking/crates/ov_cli/src/main.rs:6276` `match field_delete.command {`
- `sources/openviking/crates/ov_cli/src/main.rs:6277` `Commands::Privacy { action } => match *action {`
- `sources/openviking/crates/ov_cli/src/main.rs:6278` `PrivacyCommands::Delete {`
- `sources/openviking/crates/ov_cli/src/main.rs:6279` `selector_or_category,`
- `sources/openviking/crates/ov_cli/src/main.rs:6280` `target_key,`
- `sources/openviking/crates/ov_cli/src/main.rs:6281` `field,`
- `sources/openviking/crates/ov_cli/src/main.rs:6282` `yes,`
- `sources/openviking/crates/ov_cli/src/main.rs:6288` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:6289` `_ => panic!("expected privacy field delete"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6290` `},`
- `sources/openviking/crates/ov_cli/src/main.rs:6291` `_ => panic!("expected privacy command"),`
- `sources/openviking/crates/ov_cli/src/main.rs:6292` `}`
- `sources/openviking/crates/ov_cli/src/main.rs:6293` `});`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/openviking_assets.rs:635`; body evidence.

- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:634` `"resource_uri": orphan.resource_uri,`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:635` `"note": "no longer in manifest; OpenViking Assets never deletes resources automatically",`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:636` `}));`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:637` `}`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:638` ``
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:639` `emit(json!({`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:920` `"! orphan: '{}' left the manifest but its resource remains ({uri}); \`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:921` `OpenViking Assets never deletes automatically",`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:922` `text("name")`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:923` `);`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:924` `}`
- `sources/openviking/crates/ov_cli/src/openviking_assets.rs:925` `"asset_planned" => {`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/output.rs:120`; body evidence.

- `sources/openviking/crates/ov_cli/src/output.rs:119` ``
- `sources/openviking/crates/ov_cli/src/output.rs:120` `if let Some(rendered) = render_session_archive(obj, compact) {`
- `sources/openviking/crates/ov_cli/src/output.rs:121` `println!("{}", append_profile_section(rendered, obj));`
- `sources/openviking/crates/ov_cli/src/output.rs:122` `return;`
- `sources/openviking/crates/ov_cli/src/output.rs:123` `}`
- `sources/openviking/crates/ov_cli/src/output.rs:124` ``
- `sources/openviking/crates/ov_cli/src/output.rs:507` `) -> Option<String> {`
- `sources/openviking/crates/ov_cli/src/output.rs:508` `if !(obj.contains_key("latest_archive_overview")`
- `sources/openviking/crates/ov_cli/src/output.rs:509` `&& obj.contains_key("pre_archive_abstracts")`
- `sources/openviking/crates/ov_cli/src/output.rs:510` `&& obj.contains_key("messages"))`
- `sources/openviking/crates/ov_cli/src/output.rs:511` `{`
- `sources/openviking/crates/ov_cli/src/output.rs:512` `return None;`
- `sources/openviking/crates/ov_cli/src/output.rs:513` `}`
- `sources/openviking/crates/ov_cli/src/output.rs:514` ``
- `sources/openviking/crates/ov_cli/src/output.rs:515` `let latest_archive_overview = obj`
- `sources/openviking/crates/ov_cli/src/output.rs:516` `.get("latest_archive_overview")`
- `sources/openviking/crates/ov_cli/src/output.rs:517` `.and_then(|v| v.as_str())`
- `sources/openviking/crates/ov_cli/src/output.rs:518` `.unwrap_or("");`
- `sources/openviking/crates/ov_cli/src/output.rs:519` `let estimated_tokens = obj`
- `sources/openviking/crates/ov_cli/src/output.rs:520` `.get("estimatedTokens")`
- `sources/openviking/crates/ov_cli/src/output.rs:535` `lines.push(format!(`
- `sources/openviking/crates/ov_cli/src/output.rs:536` `"total_archives         {}",`
- `sources/openviking/crates/ov_cli/src/output.rs:537` `stats`
- `sources/openviking/crates/ov_cli/src/output.rs:538` `.get("totalArchives")`
- `sources/openviking/crates/ov_cli/src/output.rs:539` `.map(format_value)`
- `sources/openviking/crates/ov_cli/src/output.rs:540` `.unwrap_or_else(|| "0".to_string())`
- `sources/openviking/crates/ov_cli/src/output.rs:541` `));`
- `sources/openviking/crates/ov_cli/src/output.rs:542` `lines.push(format!(`
- `sources/openviking/crates/ov_cli/src/output.rs:543` `"included_archives      {}",`
- `sources/openviking/crates/ov_cli/src/output.rs:544` `stats`
- `sources/openviking/crates/ov_cli/src/output.rs:545` `.get("includedArchives")`
- `sources/openviking/crates/ov_cli/src/output.rs:546` `.map(format_value)`
- `sources/openviking/crates/ov_cli/src/output.rs:547` `.unwrap_or_else(|| "0".to_string())`
- `sources/openviking/crates/ov_cli/src/output.rs:548` `));`
- `sources/openviking/crates/ov_cli/src/output.rs:549` `lines.push(format!(`
- `sources/openviking/crates/ov_cli/src/output.rs:550` `"dropped_archives       {}",`
- `sources/openviking/crates/ov_cli/src/output.rs:551` `stats`
- `sources/openviking/crates/ov_cli/src/output.rs:552` `.get("droppedArchives")`
- `sources/openviking/crates/ov_cli/src/output.rs:553` `.map(format_value)`
- `sources/openviking/crates/ov_cli/src/output.rs:554` `.unwrap_or_else(|| "0".to_string())`
- `sources/openviking/crates/ov_cli/src/output.rs:555` `));`
- `sources/openviking/crates/ov_cli/src/output.rs:556` `}`
- `sources/openviking/crates/ov_cli/src/output.rs:558` `lines.push(String::new());`
- `sources/openviking/crates/ov_cli/src/output.rs:559` `lines.push("latest_archive_overview".to_string());`
- `sources/openviking/crates/ov_cli/src/output.rs:560` `if latest_archive_overview.is_empty() {`
- `sources/openviking/crates/ov_cli/src/output.rs:561` `let has_abstracts = obj`
- `sources/openviking/crates/ov_cli/src/output.rs:562` `.get("pre_archive_abstracts")`
- `sources/openviking/crates/ov_cli/src/output.rs:563` `.and_then(|v| v.as_array())`
- `sources/openviking/crates/ov_cli/src/output.rs:564` `.map(|items| !items.is_empty())`
- `sources/openviking/crates/ov_cli/src/output.rs:565` `.unwrap_or(false);`
- `sources/openviking/crates/ov_cli/src/output.rs:566` `if !has_abstracts {`
- `sources/openviking/crates/ov_cli/src/output.rs:571` `} else {`
- `sources/openviking/crates/ov_cli/src/output.rs:572` `lines.push(latest_archive_overview.to_string());`
- `sources/openviking/crates/ov_cli/src/output.rs:573` `}`
- `sources/openviking/crates/ov_cli/src/output.rs:574` ``
- `sources/openviking/crates/ov_cli/src/output.rs:575` `if let Some(items) = obj.get("pre_archive_abstracts").and_then(|v| v.as_array()) {`
- `sources/openviking/crates/ov_cli/src/output.rs:576` `lines.push(String::new());`
- `sources/openviking/crates/ov_cli/src/output.rs:577` `lines.push(format!("pre_archive_abstracts ({})", items.len()));`
- `sources/openviking/crates/ov_cli/src/output.rs:578` `if items.is_empty() {`
- `sources/openviking/crates/ov_cli/src/output.rs:579` `lines.push("(empty)".to_string());`
- `sources/openviking/crates/ov_cli/src/output.rs:580` `} else if let Some(table) = format_array_to_table(items, compact) {`
- `sources/openviking/crates/ov_cli/src/output.rs:581` `lines.push(table.trim_end().to_string());`
- `sources/openviking/crates/ov_cli/src/output.rs:600` ``
- `sources/openviking/crates/ov_cli/src/output.rs:601` `fn render_session_archive(`
- `sources/openviking/crates/ov_cli/src/output.rs:602` `obj: &serde_json::Map<String, serde_json::Value>,`
- `sources/openviking/crates/ov_cli/src/output.rs:603` `compact: bool,`
- `sources/openviking/crates/ov_cli/src/output.rs:604` `) -> Option<String> {`
- `sources/openviking/crates/ov_cli/src/output.rs:605` `if !(obj.contains_key("archive_id")`
- `sources/openviking/crates/ov_cli/src/output.rs:606` `&& obj.contains_key("overview")`
- `sources/openviking/crates/ov_cli/src/output.rs:607` `&& obj.contains_key("messages"))`
- `sources/openviking/crates/ov_cli/src/output.rs:608` `{`
- `sources/openviking/crates/ov_cli/src/output.rs:609` `return None;`
- `sources/openviking/crates/ov_cli/src/output.rs:611` ``
- `sources/openviking/crates/ov_cli/src/output.rs:612` `let archive_id = obj.get("archive_id").and_then(|v| v.as_str()).unwrap_or("");`
- `sources/openviking/crates/ov_cli/src/output.rs:613` `let abstract_text = obj.get("abstract").and_then(|v| v.as_str()).unwrap_or("");`
- `sources/openviking/crates/ov_cli/src/output.rs:614` `let overview = obj.get("overview").and_then(|v| v.as_str()).unwrap_or("");`
- `sources/openviking/crates/ov_cli/src/output.rs:615` ``
- `sources/openviking/crates/ov_cli/src/output.rs:616` `let mut lines: Vec<String> = Vec::new();`
- `sources/openviking/crates/ov_cli/src/output.rs:617` `lines.push(format!(`
- `sources/openviking/crates/ov_cli/src/output.rs:618` `"archive_id             {}",`
- `sources/openviking/crates/ov_cli/src/output.rs:619` `if archive_id.is_empty() {`
- `sources/openviking/crates/ov_cli/src/output.rs:620` `"(none)"`
- `sources/openviking/crates/ov_cli/src/output.rs:621` `} else {`
- `sources/openviking/crates/ov_cli/src/output.rs:622` `archive_id`
- `sources/openviking/crates/ov_cli/src/output.rs:623` `}`
- `sources/openviking/crates/ov_cli/src/output.rs:624` `));`
- `sources/openviking/crates/ov_cli/src/output.rs:625` `lines.push(format!(`
- `sources/openviking/crates/ov_cli/src/output.rs:626` `"abstract               {}",`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/tui/app.rs:578`; body evidence.

- `sources/openviking/crates/ov_cli/src/tui/app.rs:577` `/// Returns true if deletion was initiated (not whether it succeeded)`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:578` `pub async fn delete_selected_uri(&mut self) {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:579` `if let Some(selected_uri) = self.tree.selected_uri() {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:580` `let is_dir = self.tree.selected_is_dir().unwrap_or(false);`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:581` `self.delete_uri(selected_uri.to_string(), is_dir).await;`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:582` `} else {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:583` `self.set_status_message("Nothing selected to delete".to_string());`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:584` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:585` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:586` ``
- `sources/openviking/crates/ov_cli/src/tui/app.rs:587` `/// Delete a specific URI`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:588` `/// No return value - success/failure is communicated via status messages`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:589` `pub async fn delete_uri(&mut self, selected_uri: String, is_dir: bool) {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:590` `if !self.tree.allow_deletion(&selected_uri) {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:591` `self.set_error_message("Cannot delete root and scope directories".to_string());`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:592` `return;`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:593` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:594` ``
- `sources/openviking/crates/ov_cli/src/tui/app.rs:595` `let client = self.client.clone();`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:628` `Ok(_) => {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:629` `self.set_status_message(format!("Deleted: {}", selected_uri));`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:630` ``
- `sources/openviking/crates/ov_cli/src/tui/app.rs:631` `// Remove deleted node from expanded nodes`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:632` `let mut expanded_nodes = expanded_nodes;`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:633` `expanded_nodes.retain(|uri| uri != &selected_uri);`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:641` `Err(e) => {`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:642` `self.set_status_message(format!("Delete failed: {}", e));`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:643` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:644` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:645` `}`
- `sources/openviking/crates/ov_cli/src/tui/app.rs:646` `}`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/tui/event.rs:96`; body evidence.

- `sources/openviking/crates/ov_cli/src/tui/event.rs:95` `// First check if deletion is allowed`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:96` `if !app.tree.allow_deletion(&selected_uri) {`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:97` `app.set_error_message("Cannot delete root and scope directories".to_string());`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:98` `return;`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:99` `}`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:100` ``
- `sources/openviking/crates/ov_cli/src/tui/event.rs:101` `// Create confirmation`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:102` `let message = format!(`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:103` `"Delete {}? (y/n): {}",`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:104` `if is_dir { "directory" } else { "file" },`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:105` `selected_uri`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:106` `);`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:107` ``
- `sources/openviking/crates/ov_cli/src/tui/event.rs:109` `Box::pin(async move {`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:110` `app.delete_selected_uri().await;`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:111` `})`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:112` `});`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:113` `} else {`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:114` `app.set_status_message("Nothing selected to delete".to_string());`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:115` `}`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:116` `}`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:117` `KeyCode::Char('r') => {`
- `sources/openviking/crates/ov_cli/src/tui/event.rs:118` `app.reload_entire_tree().await;`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/tui/tree.rs:337`; body evidence.

- `sources/openviking/crates/ov_cli/src/tui/tree.rs:336` ``
- `sources/openviking/crates/ov_cli/src/tui/tree.rs:337` `pub fn allow_deletion(&self, selected_uri: &str) -> bool {`
- `sources/openviking/crates/ov_cli/src/tui/tree.rs:338` `selected_uri != "/"`
- `sources/openviking/crates/ov_cli/src/tui/tree.rs:339` `&& !Self::ROOT_SCOPES`
- `sources/openviking/crates/ov_cli/src/tui/tree.rs:340` `.iter()`
- `sources/openviking/crates/ov_cli/src/tui/tree.rs:341` `.any(|s| selected_uri == format!("viking://{}", s))`

**operation and route call sites**, body `sources/openviking/crates/ov_cli/src/tui/ui.rs:371`; body evidence.

- `sources/openviking/crates/ov_cli/src/tui/ui.rs:370` `),`
- `sources/openviking/crates/ov_cli/src/tui/ui.rs:371` `Span::raw(":delete  "),`
- `sources/openviking/crates/ov_cli/src/tui/ui.rs:372` `Span::styled(`
- `sources/openviking/crates/ov_cli/src/tui/ui.rs:373` `"r",`
- `sources/openviking/crates/ov_cli/src/tui/ui.rs:374` `Style::default()`
- `sources/openviking/crates/ov_cli/src/tui/ui.rs:375` `.fg(Color::Yellow)`

**_validate_forget_memory_uri**, body `sources/openviking/examples/hermes-plugin/__init__.py:616`; implementation/call-through.

- `sources/openviking/examples/hermes-plugin/__init__.py:615` `def _validate_forget_memory_uri(raw_uri: Any, *, user_space: Optional[str] = None) -> tuple[Optional[str], Optional[str]]:`
- `sources/openviking/examples/hermes-plugin/__init__.py:616` `uri = raw_uri.strip() if isinstance(raw_uri, str) else ""`

**delete**, body `sources/openviking/examples/hermes-plugin/__init__.py:433`; implementation/call-through.

- `sources/openviking/examples/hermes-plugin/__init__.py:432` `def delete(self, path: str, **kwargs) -> dict:`
- `sources/openviking/examples/hermes-plugin/__init__.py:433` `return self._request("delete", path, kwargs)`

**_tool_forget**, body `sources/openviking/examples/hermes-plugin/__init__.py:3345`; implementation/call-through.

- `sources/openviking/examples/hermes-plugin/__init__.py:3343` `def _tool_forget(self, args: dict) -> str:`
- `sources/openviking/examples/hermes-plugin/__init__.py:3349` `result = self._unwrap_result(client.delete("/api/v1/fs", params={"uri": uri, "recursive": False}))`
- `sources/openviking/examples/hermes-plugin/__init__.py:3351` `payload = {"status": "deleted", "uri": result.get("uri") or uri,`

**invalidate**, body `sources/openviking/openviking/config/embedding.py:140`; implementation/call-through.

- `sources/openviking/openviking/config/embedding.py:139` `async def invalidate(self, account_id: str) -> None:`
- `sources/openviking/openviking/config/embedding.py:140` `async with self._account_locks.acquire(account_id):`

**delete_account**, body `sources/openviking/openviking/config/manager.py:376`; implementation/call-through.

- `sources/openviking/openviking/config/manager.py:374` `async def delete_account(self, account_id: str) -> ConfigChangeEvent:`
- `sources/openviking/openviking/config/manager.py:376` `scope = ConfigScope.account(account_id)`

**_delete_account**, body `sources/openviking/openviking/config/manager.py:380`; implementation/call-through.

- `sources/openviking/openviking/config/manager.py:379` `async def _delete_account(self, scope: ConfigScope) -> ConfigChangeEvent:`
- `sources/openviking/openviking/config/manager.py:382` `await self._source.delete(scope)`

**delete**, body `sources/openviking/openviking/config/source/base.py:72`; stub.

- `sources/openviking/openviking/config/source/base.py:66` `async def delete(self, scope: ConfigScope) -> None:`
- `sources/openviking/openviking/config/source/base.py:72` `...`

**delete**, body `sources/openviking/openviking/config/source/file_source.py:171`; implementation/call-through.

- `sources/openviking/openviking/config/source/file_source.py:170` `async def delete(self, scope: ConfigScope) -> None:`
- `sources/openviking/openviking/config/source/file_source.py:171` `path = self._path(scope)`

**delete**, body `sources/openviking/openviking/config/source/memory_source.py:40`; implementation/call-through.

- `sources/openviking/openviking/config/source/memory_source.py:39` `async def delete(self, scope: ConfigScope) -> None:`
- `sources/openviking/openviking/config/source/memory_source.py:40` `async with self._lock:`

**_validate_subtree_delete**, body `sources/openviking/openviking/config/validate.py:278`; implementation/call-through.

- `sources/openviking/openviking/config/validate.py:270` `def _validate_subtree_delete(`
- `sources/openviking/openviking/config/validate.py:278` `nested_model = _model_at_path(root_model, path)`

**_invalidate**, body `sources/openviking/openviking/config/vlm.py:367`; implementation/call-through.

- `sources/openviking/openviking/config/vlm.py:366` `def _invalidate(self, matches: Callable[[_RequestKey], bool]) -> None:`
- `sources/openviking/openviking/config/vlm.py:367` `with self._lock:`

**rm**, body `sources/openviking/openviking/eval/recorder/recording_client.py:152`; implementation/call-through.

- `sources/openviking/openviking/eval/recorder/recording_client.py:150` `def rm(self, path: str, recursive: bool = False) -> Dict[str, Any]:`
- `sources/openviking/openviking/eval/recorder/recording_client.py:152` `return self._wrap_operation("rm", path, recursive)`

**delete**, body `sources/openviking/openviking/eval/recorder/wrapper.py:336`; implementation/call-through.

- `sources/openviking/openviking/eval/recorder/wrapper.py:334` `async def delete(self, collection: str, ids: List[str]) -> int:`
- `sources/openviking/openviking/eval/recorder/wrapper.py:339` `result = await self._db.delete(ids)`
- `sources/openviking/openviking/eval/recorder/wrapper.py:341` `self._record("delete", request, result, latency_ms)`
- `sources/openviking/openviking/eval/recorder/wrapper.py:345` `self._record("delete", request, None, latency_ms, False, str(e))`

**delete**, body `sources/openviking/openviking/ingest/cursor_store.py:299`; implementation/call-through.

- `sources/openviking/openviking/ingest/cursor_store.py:298` `def delete(self, harness: str, native_session_id: str) -> None:`
- `sources/openviking/openviking/ingest/cursor_store.py:300` `"DELETE FROM ingest_cursor WHERE harness = ? AND native_session_id = ?",`

**delete**, body `sources/openviking/openviking/ingest/replay.py:101`; implementation/call-through.

- `sources/openviking/openviking/ingest/replay.py:100` `async def delete(self, ov_session_id: str) -> None:`
- `sources/openviking/openviking/ingest/replay.py:102` `await self._client.delete_session(ov_session_id)`

**gauge_delete_matching**, body `sources/openviking/openviking/metrics/collectors/base.py:181`; implementation/call-through.

- `sources/openviking/openviking/metrics/collectors/base.py:173` `def gauge_delete_matching(`
- `sources/openviking/openviking/metrics/collectors/base.py:181` `if not metric_supports_account_dimension(name):`

**delete_gauge_series_for_labels**, body `sources/openviking/openviking/metrics/collectors/base.py:603`; implementation/call-through.

- `sources/openviking/openviking/metrics/collectors/base.py:594` `def delete_gauge_series_for_labels(`
- `sources/openviking/openviking/metrics/collectors/base.py:603` `for value in values:`

**record_archive**, body `sources/openviking/openviking/metrics/collectors/session.py:90`; implementation/call-through.

- `sources/openviking/openviking/metrics/collectors/session.py:88` `def record_archive(self, registry, *, status: str) -> None:`
- `sources/openviking/openviking/metrics/collectors/session.py:91` `self.ARCHIVE_TOTAL,`

**gauge_delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:399`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:391` `def gauge_delete_matching(self, name: str, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:403` `family.delete_matching(match_labels=match_labels)`

**counter_delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:407`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:405` `def counter_delete_matching(self, name: str, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:410` `family.delete_matching(match_labels=match_labels)`

**histogram_delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:414`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:412` `def histogram_delete_matching(self, name: str, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:417` `family.delete_matching(match_labels=match_labels)`

**delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:519`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:517` `def delete_matching(self, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:519` `normalized = normalize_labels(match_labels)`

**delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:609`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:607` `def delete_matching(self, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:609` `normalized = normalize_labels(match_labels)`

**delete_matching**, body `sources/openviking/openviking/metrics/core/registry.py:705`; implementation/call-through.

- `sources/openviking/openviking/metrics/core/registry.py:703` `def delete_matching(self, *, match_labels: Mapping[str, str]) -> None:`
- `sources/openviking/openviking/metrics/core/registry.py:705` `normalized = normalize_labels(match_labels)`

**record_archive**, body `sources/openviking/openviking/metrics/datasources/session.py:33`; implementation/call-through.

- `sources/openviking/openviking/metrics/datasources/session.py:30` `@staticmethod`
- `sources/openviking/openviking/metrics/datasources/session.py:31` `def record_archive(*, status: str) -> None:`
- `sources/openviking/openviking/metrics/datasources/session.py:34` `"session.archive",`

**delete_codex_auth_store**, body `sources/openviking/openviking/models/vlm/backends/codex_auth.py:323`; implementation/call-through.

- `sources/openviking/openviking/models/vlm/backends/codex_auth.py:322` `def delete_codex_auth_store() -> bool:`
- `sources/openviking/openviking/models/vlm/backends/codex_auth.py:327` `path.unlink()`

**_delete_remote_file**, body `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:316`; implementation/call-through.

- `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:315` `async def _delete_remote_file(vlm: VolcEngineVLM, file_id: str) -> None:`
- `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:320` `client.files.delete(file_id, extra_headers=vlm.extra_headers),`

**bounded_delete**, body `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:319`; implementation/call-through.

- `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:318` `async def bounded_delete() -> None:`
- `sources/openviking/openviking/models/vlm/backends/volcengine_media.py:320` `client.files.delete(file_id, extra_headers=vlm.extra_headers),`

**reset**, body `sources/openviking/openviking/models/vlm/token_usage.py:39`; implementation/call-through.

- `sources/openviking/openviking/models/vlm/token_usage.py:37` `def reset(self) -> None:`
- `sources/openviking/openviking/models/vlm/token_usage.py:39` `self.prompt_tokens = 0`

**reset**, body `sources/openviking/openviking/models/vlm/token_usage.py:211`; implementation/call-through.

- `sources/openviking/openviking/models/vlm/token_usage.py:208` `@_locked`
- `sources/openviking/openviking/models/vlm/token_usage.py:209` `def reset(self) -> None:`
- `sources/openviking/openviking/models/vlm/token_usage.py:211` `self._usage_by_model.clear()`

**clear**, body `sources/openviking/openviking/observability/events.py:80`; implementation/call-through.

- `sources/openviking/openviking/observability/events.py:78` `def clear(self) -> None:`
- `sources/openviking/openviking/observability/events.py:80` `with self._lock:`

**clear**, body `sources/openviking/openviking/observability/http_observability_middleware.py:204`; implementation/call-through.

- `sources/openviking/openviking/observability/http_observability_middleware.py:202` `def clear(self) -> None:`
- `sources/openviking/openviking/observability/http_observability_middleware.py:204` `for i in range(self._num_shards):`

**delete_data**, body `sources/openviking/openviking/observability/usage_audit/runtime.py:41`; implementation/call-through.

- `sources/openviking/openviking/observability/usage_audit/runtime.py:39` `async def delete_data(self, *, account_id: str, user_id: str | None = None) -> dict[str, int]:`
- `sources/openviking/openviking/observability/usage_audit/runtime.py:42` `return await self.store.delete_data(account_id=account_id, user_id=user_id)`

**delete_data**, body `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:139`; implementation/call-through.

- `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:137` `async def delete_data(self, *, account_id: str, user_id: str | None = None) -> dict[str, int]:`
- `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:139` `async with self._lock:`

**_delete_data_sync**, body `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:143`; implementation/call-through.

- `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:142` `def _delete_data_sync(self, account_id: str, user_id: str | None = None) -> dict[str, int]:`
- `sources/openviking/openviking/observability/usage_audit/sqlite_store.py:156` `f"DELETE FROM {table} WHERE {predicate}",`

**delete_data**, body `sources/openviking/openviking/observability/usage_audit/store.py:29`; implementation/call-through.

- `sources/openviking/openviking/observability/usage_audit/store.py:29` `async def delete_data(self, *, account_id: str, user_id: str | None = None) -> dict[str, int]:`

**_cleanup_archive**, body `sources/openviking/openviking/parse/accessors/git_accessor.py:532`; implementation/call-through.

- `sources/openviking/openviking/parse/accessors/git_accessor.py:531` `def _cleanup_archive():`
- `sources/openviking/openviking/parse/accessors/git_accessor.py:533` `os.unlink(zip_path)`

**_cleanup_archive**, body `sources/openviking/openviking/parse/accessors/git_accessor.py:625`; implementation/call-through.

- `sources/openviking/openviking/parse/accessors/git_accessor.py:624` `def _cleanup_archive():`
- `sources/openviking/openviking/parse/accessors/git_accessor.py:626` `os.unlink(zip_path)`

**clear**, body `sources/openviking/openviking/parse/accessors/registry.py:187`; implementation/call-through.

- `sources/openviking/openviking/parse/accessors/registry.py:185` `def clear(self) -> None:`
- `sources/openviking/openviking/parse/accessors/registry.py:187` `self._accessors.clear()`

**delete**, body `sources/openviking/openviking/privacy/service.py:81`; implementation/call-through.

- `sources/openviking/openviking/privacy/service.py:73` `async def delete(`
- `sources/openviking/openviking/privacy/service.py:81` `root_uri = self.get_config_root(ctx, category, target_key)`

**delete_field**, body `sources/openviking/openviking/privacy/service.py:421`; implementation/call-through.

- `sources/openviking/openviking/privacy/service.py:408` `async def delete_field(`
- `sources/openviking/openviking/privacy/service.py:421` `validate_privacy_field_name(field, location="field")`

**rm**, body `sources/openviking/openviking/pyagfs/async_client.py:212`; implementation/call-through.

- `sources/openviking/openviking/pyagfs/async_client.py:198` `async def rm(`
- `sources/openviking/openviking/pyagfs/async_client.py:212` `kwargs: Dict[str, Any] = {}`

**rm**, body `sources/openviking/openviking/pyagfs/protocols.py:81`; implementation/call-through.

- `sources/openviking/openviking/pyagfs/protocols.py:81` `def rm(self, path: str, recursive: bool = False, force: bool = True) -> Dict[str, Any]:`

**delete_task**, body `sources/openviking/openviking/resource/watch_manager.py:876`; implementation/call-through.

- `sources/openviking/openviking/resource/watch_manager.py:855` `async def delete_task(`
- `sources/openviking/openviking/resource/watch_manager.py:876` `while True:`

**delete_tasks**, body `sources/openviking/openviking/resource/watch_scheduler.py:188`; implementation/call-through.

- `sources/openviking/openviking/resource/watch_scheduler.py:186` `async def delete_tasks(self, account_id: str, user_id: str | None = None) -> None:`
- `sources/openviking/openviking/resource/watch_scheduler.py:194` `await manager.delete_task(watch.task_id, account_id, actor_user_id, Role.ROOT)`

**reset**, body `sources/openviking/openviking/retrieve/retrieval_stats.py:162`; implementation/call-through.

- `sources/openviking/openviking/retrieve/retrieval_stats.py:160` `def reset(self) -> None:`
- `sources/openviking/openviking/retrieve/retrieval_stats.py:162` `with self._lock:`

**delete_account**, body `sources/openviking/openviking/server/account_stores/base.py:47`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:46` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:47` `async def delete_account(self, account_id: str) -> None: ...`

**delete_group**, body `sources/openviking/openviking/server/account_stores/base.py:119`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:118` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:119` `async def delete_group(self, account_id: str, group_id: str) -> None: ...`

**begin_deletion**, body `sources/openviking/openviking/server/account_stores/base.py:137`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:128` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:129` `async def begin_deletion(`
- `sources/openviking/openviking/server/account_stores/base.py:137` `) -> tuple[DeletionRecord, bool]: ...`

**replace_deletion_task**, body `sources/openviking/openviking/server/account_stores/base.py:149`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:139` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:140` `async def replace_deletion_task(`
- `sources/openviking/openviking/server/account_stores/base.py:149` `) -> DeletionRecord: ...`

**finish_deletion**, body `sources/openviking/openviking/server/account_stores/base.py:152`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:151` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:152` `async def finish_deletion(self, account_id: str, user_id: str | None, task_id: str) -> bool: ...`

**get_deletion**, body `sources/openviking/openviking/server/account_stores/base.py:157`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:154` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:155` `async def get_deletion(`
- `sources/openviking/openviking/server/account_stores/base.py:157` `) -> DeletionRecord | None: ...`

**iter_deletions**, body `sources/openviking/openviking/server/account_stores/base.py:162`; stub.

- `sources/openviking/openviking/server/account_stores/base.py:159` `@abstractmethod`
- `sources/openviking/openviking/server/account_stores/base.py:160` `async def iter_deletions(`
- `sources/openviking/openviking/server/account_stores/base.py:162` `) -> list[tuple[str, str | None, DeletionRecord]]: ...`

**_deletion**, body `sources/openviking/openviking/server/account_stores/mysql.py:123`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:122` `def _deletion(record) -> DeletionRecord | None:`
- `sources/openviking/openviking/server/account_stores/mysql.py:123` `if record.deletion_task_id is None:`

**delete_account**, body `sources/openviking/openviking/server/account_stores/mysql.py:586`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:585` `async def delete_account(self, account_id: str) -> None:`
- `sources/openviking/openviking/server/account_stores/mysql.py:587` `await session.delete(await self._lock_account(session, account_id))`

**begin_deletion**, body `sources/openviking/openviking/server/account_stores/mysql.py:790`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:781` `async def begin_deletion(`
- `sources/openviking/openviking/server/account_stores/mysql.py:790` `_validate_length(owner_account_id, "owner_account_id")`

**replace_deletion_task**, body `sources/openviking/openviking/server/account_stores/mysql.py:820`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:810` `async def replace_deletion_task(`
- `sources/openviking/openviking/server/account_stores/mysql.py:820` `_validate_length(owner_account_id, "owner_account_id")`

**finish_deletion**, body `sources/openviking/openviking/server/account_stores/mysql.py:841`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:840` `async def finish_deletion(self, account_id: str, user_id: str | None, task_id: str) -> bool:`
- `sources/openviking/openviking/server/account_stores/mysql.py:849` `await session.delete(account)`
- `sources/openviking/openviking/server/account_stores/mysql.py:859` `await session.delete(user)`

**get_deletion**, body `sources/openviking/openviking/server/account_stores/mysql.py:865`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:862` `async def get_deletion(`
- `sources/openviking/openviking/server/account_stores/mysql.py:865` `if user_id is None:`

**iter_deletions**, body `sources/openviking/openviking/server/account_stores/mysql.py:899`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:898` `async def iter_deletions(self) -> list[tuple[str, str | None, DeletionRecord]]:`
- `sources/openviking/openviking/server/account_stores/mysql.py:899` `async with self._session_factory() as session:`

**delete_group**, body `sources/openviking/openviking/server/account_stores/mysql.py:1136`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/mysql.py:1135` `async def delete_group(self, account_id: str, group_id: str) -> None:`
- `sources/openviking/openviking/server/account_stores/mysql.py:1151` `await session.delete(group)`

**_account_deletion_key**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:297`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:296` `def _account_deletion_key(self, account_id: str) -> str:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:297` `return self._key(account_id, "deletion:account")`

**_user_deletion_key**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:300`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:299` `def _user_deletion_key(self, account_id: str, user_id: str) -> str:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:300` `return self._key(account_id, f"deletion:user:{_key_part(user_id)}")`

**_delete**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:324`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:323` `async def _delete(self, *keys: str) -> None:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:326` `await self._redis.delete(*keys)`

**_delete_and_unindex**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:458`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:457` `async def _delete_and_unindex(self, account_id: str, *keys: str) -> None:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:458` `assert self._redis is not None`

**_invalidate**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:468`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:467` `async def _invalidate(self, account_id: str, *keys: str) -> None:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:471` `logger.exception("Failed to invalidate Redis account cache")`

**_invalidate_after_lock_timeout**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:507`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:500` `async def _invalidate_after_lock_timeout(`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:516` `await self._invalidate(account_id, *keys)`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:525` `await self._invalidate(account_id, *keys)`

**get_deletion**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:738`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:735` `async def get_deletion(`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:738` `account_deletion = await self._read_through(`

**iter_deletions**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:753`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:752` `async def iter_deletions(self) -> list[tuple[str, str | None, DeletionRecord]]:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:753` `return await self._mysql.iter_deletions()`

**delete_account**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:788`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:787` `async def delete_account(self, account_id: str) -> None:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:790` `lambda: self._mysql.delete_account(account_id),`

**delete_group**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:864`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:863` `async def delete_group(self, account_id: str, group_id: str) -> None:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:866` `lambda: self._mysql.delete_group(account_id, group_id),`

**begin_deletion**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:890`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:887` `async def begin_deletion(`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:890` `return await self._mutate_account(`

**replace_deletion_task**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:903`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:900` `async def replace_deletion_task(`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:903` `return await self._mutate_account(`

**finish_deletion**, body `sources/openviking/openviking/server/account_stores/redis_mysql.py:914`; implementation/call-through.

- `sources/openviking/openviking/server/account_stores/redis_mysql.py:913` `async def finish_deletion(self, account_id: str, user_id: str | None, task_id: str) -> bool:`
- `sources/openviking/openviking/server/account_stores/redis_mysql.py:914` `return await self._mutate_account(`

**_delete_account_identity**, body `sources/openviking/openviking/server/api_keys/legacy.py:513`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:511` `async def _delete_account_identity(self, account_id: str) -> None:`
- `sources/openviking/openviking/server/api_keys/legacy.py:513` `if account_id not in self._accounts:`

**_begin_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:745`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:735` `async def _begin_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:745` `async with self._reload_lock, self._deletion_lock:`

**_replace_deletion_task**, body `sources/openviking/openviking/server/api_keys/legacy.py:808`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:797` `async def _replace_deletion_task(`
- `sources/openviking/openviking/server/api_keys/legacy.py:808` `async with self._reload_lock, self._deletion_lock:`

**_finish_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:852`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:848` `async def _finish_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:852` `async with self._reload_lock, self._deletion_lock:`

**_get_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:895`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:892` `def _get_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:895` `account = self._accounts.get(account_id)`

**_iter_deletions**, body `sources/openviking/openviking/server/api_keys/legacy.py:905`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:904` `def _iter_deletions(self) -> list[tuple[str, str | None, dict]]:`
- `sources/openviking/openviking/server/api_keys/legacy.py:905` `return [`

**_delete_group**, body `sources/openviking/openviking/server/api_keys/legacy.py:1133`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1132` `async def _delete_group(self, account_id: str, group_id: str) -> None:`
- `sources/openviking/openviking/server/api_keys/legacy.py:1133` `async with self._reload_lock:`

**delete_account**, body `sources/openviking/openviking/server/api_keys/legacy.py:1529`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1528` `async def delete_account(self, account_id: str) -> None:`
- `sources/openviking/openviking/server/api_keys/legacy.py:1529` `async with self._mutation_lock:`

**begin_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:1589`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1580` `async def begin_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:1589` `async with self._mutation_lock:`

**replace_deletion_task**, body `sources/openviking/openviking/server/api_keys/legacy.py:1611`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1601` `async def replace_deletion_task(`
- `sources/openviking/openviking/server/api_keys/legacy.py:1611` `async with self._mutation_lock:`

**finish_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:1624`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1621` `async def finish_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:1624` `async with self._mutation_lock:`

**get_deletion**, body `sources/openviking/openviking/server/api_keys/legacy.py:1630`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1627` `async def get_deletion(`
- `sources/openviking/openviking/server/api_keys/legacy.py:1630` `return self._get_deletion(account_id) or (`

**iter_deletions**, body `sources/openviking/openviking/server/api_keys/legacy.py:1637`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1634` `async def iter_deletions(`
- `sources/openviking/openviking/server/api_keys/legacy.py:1637` `return self._iter_deletions()`

**delete_group**, body `sources/openviking/openviking/server/api_keys/legacy.py:1677`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/legacy.py:1676` `async def delete_group(self, account_id: str, group_id: str) -> None:`
- `sources/openviking/openviking/server/api_keys/legacy.py:1677` `await self._refresh_accounts_for_management_read()`

**delete_account**, body `sources/openviking/openviking/server/api_keys/new.py:152`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:151` `async def delete_account(self, account_id: str) -> None:`
- `sources/openviking/openviking/server/api_keys/new.py:152` `await self._store.delete_account(account_id)`

**begin_deletion**, body `sources/openviking/openviking/server/api_keys/new.py:201`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:200` `async def begin_deletion(self, account_id, user_id, **owner):`
- `sources/openviking/openviking/server/api_keys/new.py:201` `return await self._store.begin_deletion(account_id, user_id, **owner)`

**replace_deletion_task**, body `sources/openviking/openviking/server/api_keys/new.py:204`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:203` `async def replace_deletion_task(self, account_id, user_id, **owner):`
- `sources/openviking/openviking/server/api_keys/new.py:204` `return await self._store.replace_deletion_task(account_id, user_id, **owner)`

**finish_deletion**, body `sources/openviking/openviking/server/api_keys/new.py:207`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:206` `async def finish_deletion(self, account_id, user_id, task_id):`
- `sources/openviking/openviking/server/api_keys/new.py:207` `return await self._store.finish_deletion(account_id, user_id, task_id)`

**get_deletion**, body `sources/openviking/openviking/server/api_keys/new.py:210`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:209` `async def get_deletion(self, account_id, user_id=None):`
- `sources/openviking/openviking/server/api_keys/new.py:210` `return await self._store.get_deletion(account_id, user_id)`

**iter_deletions**, body `sources/openviking/openviking/server/api_keys/new.py:213`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:212` `async def iter_deletions(self):`
- `sources/openviking/openviking/server/api_keys/new.py:213` `return await self._store.iter_deletions()`

**delete_group**, body `sources/openviking/openviking/server/api_keys/new.py:280`; implementation/call-through.

- `sources/openviking/openviking/server/api_keys/new.py:279` `async def delete_group(self, account_id, group_id):`
- `sources/openviking/openviking/server/api_keys/new.py:280` `await self._store.delete_group(account_id, group_id)`

**forget**, body `sources/openviking/openviking/server/mcp_endpoint.py:2185`; implementation/call-through.

- `sources/openviking/openviking/server/mcp_endpoint.py:2178` `@_mcp_error_results()`
- `sources/openviking/openviking/server/mcp_endpoint.py:2179` `@mcp.tool(annotations=_RETRY_SAFE_DESTRUCTIVE_TOOL_ANNOTATIONS)`
- `sources/openviking/openviking/server/mcp_endpoint.py:2180` `async def forget(uri: str, recursive: bool = False) -> str:`
- `sources/openviking/openviking/server/mcp_endpoint.py:2185` `service = get_service()`

**delete_pending_authorization**, body `sources/openviking/openviking/server/oauth/storage.py:744`; implementation/call-through.

- `sources/openviking/openviking/server/oauth/storage.py:743` `async def delete_pending_authorization(self, pending_id: str) -> None:`
- `sources/openviking/openviking/server/oauth/storage.py:747` `"DELETE FROM oauth_pending_authorizations WHERE pending_id = ?",`

**delete_acl**, body `sources/openviking/openviking/server/routers/acl.py:66`; implementation/call-through.

- `sources/openviking/openviking/server/routers/acl.py:61` `@router.delete("")`
- `sources/openviking/openviking/server/routers/acl.py:62` `async def delete_acl(`
- `sources/openviking/openviking/server/routers/acl.py:67` `result = await get_service().fs.delete_acl(uri, ctx=_ctx)`

**delete_account**, body `sources/openviking/openviking/server/routers/admin.py:582`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:574` `@router.delete("/accounts/{account_id}", status_code=202)`
- `sources/openviking/openviking/server/routers/admin.py:575` `@require_auth_root`
- `sources/openviking/openviking/server/routers/admin.py:576` `async def delete_account(`
- `sources/openviking/openviking/server/routers/admin.py:585` `result = await deletion_service.delete(account_id, actor=ctx)`

**patch_account_settings**, body `sources/openviking/openviking/server/routers/admin.py:621`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:608` `@router.patch("/accounts/{account_id}/settings", deprecated=True)`
- `sources/openviking/openviking/server/routers/admin.py:609` `@require_auth_root_or_admin`
- `sources/openviking/openviking/server/routers/admin.py:610` `async def patch_account_settings(`
- `sources/openviking/openviking/server/routers/admin.py:621` `_check_account_access(ctx, account_id)`

**patch_account_configuration**, body `sources/openviking/openviking/server/routers/admin.py:782`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:773` `@router.patch("/accounts/{account_id}/configuration")`
- `sources/openviking/openviking/server/routers/admin.py:774` `@require_auth_root_or_admin`
- `sources/openviking/openviking/server/routers/admin.py:775` `async def patch_account_configuration(`
- `sources/openviking/openviking/server/routers/admin.py:782` `_check_account_access(ctx, account_id)`

**patch_cluster_configuration**, body `sources/openviking/openviking/server/routers/admin.py:917`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:906` `@router.patch("/configuration")`
- `sources/openviking/openviking/server/routers/admin.py:907` `@require_auth_root`
- `sources/openviking/openviking/server/routers/admin.py:908` `async def patch_cluster_configuration(`
- `sources/openviking/openviking/server/routers/admin.py:917` `if dry_run and (source != "file" or body.content is None):`

**patch_user_settings**, body `sources/openviking/openviking/server/routers/admin.py:1080`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:1070` `@router.patch("/accounts/{account_id}/users/{user_id}/settings")`
- `sources/openviking/openviking/server/routers/admin.py:1071` `@require_auth_root_or_admin`
- `sources/openviking/openviking/server/routers/admin.py:1072` `async def patch_user_settings(`
- `sources/openviking/openviking/server/routers/admin.py:1080` `_check_account_access(ctx, account_id)`

**remove_user**, body `sources/openviking/openviking/server/routers/admin.py:1112`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:1103` `@router.delete("/accounts/{account_id}/users/{user_id}", status_code=202)`
- `sources/openviking/openviking/server/routers/admin.py:1104` `@require_auth_root_or_admin`
- `sources/openviking/openviking/server/routers/admin.py:1105` `async def remove_user(`
- `sources/openviking/openviking/server/routers/admin.py:1116` `result = await deletion_service.delete(account_id, user_id, actor=ctx)`

**delete_group**, body `sources/openviking/openviking/server/routers/admin.py:1202`; implementation/call-through.

- `sources/openviking/openviking/server/routers/admin.py:1194` `@router.delete("/accounts/{account_id}/groups/{group_id}")`
- `sources/openviking/openviking/server/routers/admin.py:1195` `@require_auth_root_or_admin`
- `sources/openviking/openviking/server/routers/admin.py:1196` `async def delete_group(`
- `sources/openviking/openviking/server/routers/admin.py:1203` `await _get_api_key_manager(request).delete_group(account_id, group_id)`
- `sources/openviking/openviking/server/routers/admin.py:1204` `return Response(status="ok", result={"deleted": True})`

**delete_connection**, body `sources/openviking/openviking/server/routers/bot_studio.py:181`; implementation/call-through.

- `sources/openviking/openviking/server/routers/bot_studio.py:175` `@router.delete(ACCOUNT_BOT + "/connections/{connection_id}")`
- `sources/openviking/openviking/server/routers/bot_studio.py:176` `async def delete_connection(`
- `sources/openviking/openviking/server/routers/bot_studio.py:181` `return await dispatch(ctx, "update", {"action": "delete", "revision": revision}, connection_id)`

**rm**, body `sources/openviking/openviking/server/routers/filesystem.py:306`; implementation/call-through.

- `sources/openviking/openviking/server/routers/filesystem.py:298` `@router.delete("")`
- `sources/openviking/openviking/server/routers/filesystem.py:299` `async def rm(`
- `sources/openviking/openviking/server/routers/filesystem.py:306` `service = get_service()`

**delete_privacy_selector**, body `sources/openviking/openviking/server/routers/privacy_configs.py:153`; implementation/call-through.

- `sources/openviking/openviking/server/routers/privacy_configs.py:148` `@router.delete("/selector")`
- `sources/openviking/openviking/server/routers/privacy_configs.py:149` `async def delete_privacy_selector(`
- `sources/openviking/openviking/server/routers/privacy_configs.py:161` `outcome = await privacy.delete_field(`
- `sources/openviking/openviking/server/routers/privacy_configs.py:193` `deleted = await privacy.delete(`

**delete_privacy_field**, body `sources/openviking/openviking/server/routers/privacy_configs.py:333`; implementation/call-through.

- `sources/openviking/openviking/server/routers/privacy_configs.py:326` `@router.delete("/{category}/{target_key}/field")`
- `sources/openviking/openviking/server/routers/privacy_configs.py:327` `async def delete_privacy_field(`
- `sources/openviking/openviking/server/routers/privacy_configs.py:337` `outcome = await privacy.delete_field(`

**delete_privacy_config**, body `sources/openviking/openviking/server/routers/privacy_configs.py:444`; implementation/call-through.

- `sources/openviking/openviking/server/routers/privacy_configs.py:437` `@router.delete("/{category}/{target_key}")`
- `sources/openviking/openviking/server/routers/privacy_configs.py:438` `async def delete_privacy_config(`
- `sources/openviking/openviking/server/routers/privacy_configs.py:448` `deleted = await privacy.delete(_ctx, category, target_key)`

**get_session_archive**, body `sources/openviking/openviking/server/routers/sessions.py:535`; implementation/call-through.

- `sources/openviking/openviking/server/routers/sessions.py:528` `@router.get("/{session_id}/archives/{archive_id}")`
- `sources/openviking/openviking/server/routers/sessions.py:529` `async def get_session_archive(`
- `sources/openviking/openviking/server/routers/sessions.py:540` `result = await session.get_session_archive(archive_id)`
- `sources/openviking/openviking/server/routers/sessions.py:542` `return error_response(code="NOT_FOUND", message=f"Archive {archive_id} not found")`

**delete_session**, body `sources/openviking/openviking/server/routers/sessions.py:552`; implementation/call-through.

- `sources/openviking/openviking/server/routers/sessions.py:546` `@router.delete("/{session_id}")`
- `sources/openviking/openviking/server/routers/sessions.py:547` `async def delete_session(`
- `sources/openviking/openviking/server/routers/sessions.py:553` `await service.sessions.delete(session_id, _ctx)`

**delete_skill**, body `sources/openviking/openviking/server/routers/skills.py:1039`; implementation/call-through.

- `sources/openviking/openviking/server/routers/skills.py:1032` `@router.delete("/{skill_name}")`
- `sources/openviking/openviking/server/routers/skills.py:1033` `async def delete_skill(`
- `sources/openviking/openviking/server/routers/skills.py:1045` `privacy_deleted = await privacy.delete(_ctx, "skill", skill_name)`

**delete_ignore**, body `sources/openviking/openviking/server/routers/snapshot.py:307`; implementation/call-through.

- `sources/openviking/openviking/server/routers/snapshot.py:302` `@router.delete("/ignore")`
- `sources/openviking/openviking/server/routers/snapshot.py:303` `async def delete_ignore(`
- `sources/openviking/openviking/server/routers/snapshot.py:309` `await service.fs.delete_gitignore(ctx=_ctx)`

**patch_add_locations**, body `sources/openviking/openviking/server/routers/user_settings.py:83`; implementation/call-through.

- `sources/openviking/openviking/server/routers/user_settings.py:77` `@router.patch("/add-locations")`
- `sources/openviking/openviking/server/routers/user_settings.py:78` `async def patch_add_locations(`
- `sources/openviking/openviking/server/routers/user_settings.py:83` `viking_fs = _viking_fs()`

**delete_add_locations**, body `sources/openviking/openviking/server/routers/user_settings.py:114`; implementation/call-through.

- `sources/openviking/openviking/server/routers/user_settings.py:109` `@router.delete("/add-locations")`
- `sources/openviking/openviking/server/routers/user_settings.py:110` `async def delete_add_locations(`
- `sources/openviking/openviking/server/routers/user_settings.py:114` `await delete_user_add_targets(_viking_fs(), _ctx)`

**patch_watch_by_id**, body `sources/openviking/openviking/server/routers/watches.py:245`; implementation/call-through.

- `sources/openviking/openviking/server/routers/watches.py:227` `@router.patch("/watches/{task_id}")`
- `sources/openviking/openviking/server/routers/watches.py:228` `async def patch_watch_by_id(`
- `sources/openviking/openviking/server/routers/watches.py:245` `task = await _resolve_task(task_id, to_uri, _ctx)`

**_delete_impl**, body `sources/openviking/openviking/server/routers/watches.py:261`; implementation/call-through.

- `sources/openviking/openviking/server/routers/watches.py:260` `async def _delete_impl(target: WatchTask, ctx: RequestContext):`
- `sources/openviking/openviking/server/routers/watches.py:264` `ok = await wm.delete_task(target.task_id, account_id, user_id, role)`

**delete_watch_by_id**, body `sources/openviking/openviking/server/routers/watches.py:292`; implementation/call-through.

- `sources/openviking/openviking/server/routers/watches.py:275` `@router.delete("/watches/{task_id}")`
- `sources/openviking/openviking/server/routers/watches.py:276` `async def delete_watch_by_id(`
- `sources/openviking/openviking/server/routers/watches.py:292` `task = await _resolve_task(task_id, to_uri, _ctx)`

**delete_watch_by_uri**, body `sources/openviking/openviking/server/routers/watches.py:302`; implementation/call-through.

- `sources/openviking/openviking/server/routers/watches.py:296` `@router.delete("/watches")`
- `sources/openviking/openviking/server/routers/watches.py:297` `async def delete_watch_by_uri(`
- `sources/openviking/openviking/server/routers/watches.py:302` `task = await _resolve_task(None, to_uri, _ctx)`

**delete**, body `sources/openviking/openviking/server/routers/webdav.py:348`; implementation/call-through.

- `sources/openviking/openviking/server/routers/webdav.py:342` `@router.api_route("", methods=["DELETE"])`
- `sources/openviking/openviking/server/routers/webdav.py:343` `@router.api_route("/{resource_path:path}", methods=["DELETE"])`
- `sources/openviking/openviking/server/routers/webdav.py:344` `async def delete(`
- `sources/openviking/openviking/server/routers/webdav.py:357` `return _error(405, "Deleting the resources root is not supported")`

**clear**, body `sources/openviking/openviking/server/upload_token_store.py:199`; implementation/call-through.

- `sources/openviking/openviking/server/upload_token_store.py:197` `def clear(self) -> None:`
- `sources/openviking/openviking/server/upload_token_store.py:199` `self._store.clear()`

**_purge_expired**, body `sources/openviking/openviking/server/upload_token_store.py:202`; implementation/call-through.

- `sources/openviking/openviking/server/upload_token_store.py:201` `def _purge_expired(self) -> None:`
- `sources/openviking/openviking/server/upload_token_store.py:202` `now = time.time()`

**delete_user_config**, body `sources/openviking/openviking/server/user_config.py:224`; implementation/call-through.

- `sources/openviking/openviking/server/user_config.py:223` `async def delete_user_config(viking_fs: VikingFS, ctx: RequestContext) -> None:`
- `sources/openviking/openviking/server/user_config.py:224` `try:`

**delete_user_add_targets**, body `sources/openviking/openviking/server/user_config.py:252`; implementation/call-through.

- `sources/openviking/openviking/server/user_config.py:251` `async def delete_user_add_targets(viking_fs: VikingFS, ctx: RequestContext) -> None:`
- `sources/openviking/openviking/server/user_config.py:252` `def _clear(user_config: UserConfig) -> None:`

**_deletion_message**, body `sources/openviking/openviking/service/deletion.py:51`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:43` `def _deletion_message(`
- `sources/openviking/openviking/service/deletion.py:51` `return {`

**setup_deletion**, body `sources/openviking/openviking/service/deletion.py:514`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:505` `async def setup_deletion(`
- `sources/openviking/openviking/service/deletion.py:514` `if manager is None or service.viking_fs is None or service._queue_manager is None:`

**_is_deleting**, body `sources/openviking/openviking/service/deletion.py:131`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:130` `async def _is_deleting(self, account_id: str, user_id: str | None = None) -> bool:`
- `sources/openviking/openviking/service/deletion.py:131` `return await self._manager.get_deletion(account_id, user_id) is not None`

**delete**, body `sources/openviking/openviking/service/deletion.py:142`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:133` `async def delete(`
- `sources/openviking/openviking/service/deletion.py:142` `return await run_to_completion(lambda: self._submit(account_id, user_id, actor=actor))`

**delete_now**, body `sources/openviking/openviking/service/deletion.py:157`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:144` `async def delete_now(`
- `sources/openviking/openviking/service/deletion.py:171` `await self.delete(account_id, actor=actor)`

**_delete_account_files**, body `sources/openviking/openviking/service/deletion.py:380`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:379` `async def _delete_account_files(self, account_id: str) -> None:`
- `sources/openviking/openviking/service/deletion.py:380` `agfs = self._service.viking_fs._async_agfs`

**_delete_uploads**, body `sources/openviking/openviking/service/deletion.py:424`; implementation/call-through.

- `sources/openviking/openviking/service/deletion.py:423` `async def _delete_uploads(self, ctx: RequestContext) -> None:`
- `sources/openviking/openviking/service/deletion.py:439` `upload_id = uri.removeprefix(f"{_SHARED_UPLOAD_ROOT}/")`

**rm**, body `sources/openviking/openviking/service/fs_service.py:467`; implementation/call-through.

- `sources/openviking/openviking/service/fs_service.py:460` `async def rm(`
- `sources/openviking/openviking/service/fs_service.py:467` `async def execute() -> Optional[Dict[str, Any]]:`

**_rm**, body `sources/openviking/openviking/service/fs_service.py:493`; implementation/call-through.

- `sources/openviking/openviking/service/fs_service.py:485` `async def _rm(`
- `sources/openviking/openviking/service/fs_service.py:493` `viking_fs = self._ensure_initialized()`

**_enqueue_delete_refresh**, body `sources/openviking/openviking/service/fs_service.py:640`; implementation/call-through.

- `sources/openviking/openviking/service/fs_service.py:631` `async def _enqueue_delete_refresh(`
- `sources/openviking/openviking/service/fs_service.py:640` `semantic_config = get_openviking_config().semantic`

**delete_acl**, body `sources/openviking/openviking/service/fs_service.py:1357`; implementation/call-through.

- `sources/openviking/openviking/service/fs_service.py:1356` `async def delete_acl(self, uri: str, ctx: RequestContext) -> Dict[str, Any]:`
- `sources/openviking/openviking/service/fs_service.py:1357` `return await self._ensure_initialized().delete_acl(uri, ctx=ctx)`

**delete_gitignore**, body `sources/openviking/openviking/service/fs_service.py:1473`; implementation/call-through.

- `sources/openviking/openviking/service/fs_service.py:1470` `async def delete_gitignore(self, *, ctx: RequestContext) -> None:`
- `sources/openviking/openviking/service/fs_service.py:1474` `await viking_fs.delete_gitignore(ctx=ctx)`

**_delete_vectors**, body `sources/openviking/openviking/service/legacy_migration.py:411`; implementation/call-through.

- `sources/openviking/openviking/service/legacy_migration.py:410` `async def _delete_vectors(self, target: LegacyCleanupTarget) -> VectorMigrationResult:`
- `sources/openviking/openviking/service/legacy_migration.py:411` `vector_store = getattr(self._service, "vikingdb_manager", None)`

**delete_uri_level**, body `sources/openviking/openviking/service/reindex_executor.py:1079`; implementation/call-through.

- `sources/openviking/openviking/service/reindex_executor.py:1073` `async def delete_uri_level(self, *, uri: str, level: ContextLevel, ctx: RequestContext) -> int:`
- `sources/openviking/openviking/service/reindex_executor.py:1090` `return await service.vikingdb_manager.delete(ids, ctx=ctx)`

**_resource_deletion_memory_policy**, body `sources/openviking/openviking/service/resource_memory_link_service.py:79`; implementation/call-through.

- `sources/openviking/openviking/service/resource_memory_link_service.py:77` `def _resource_deletion_memory_policy(target_peer_id: Optional[str] = None) -> Dict[str, Any]:`
- `sources/openviking/openviking/service/resource_memory_link_service.py:79` `return _resource_memory_policy(`

**on_resource_deleted**, body `sources/openviking/openviking/service/resource_memory_link_service.py:229`; implementation/call-through.

- `sources/openviking/openviking/service/resource_memory_link_service.py:220` `async def on_resource_deleted(`
- `sources/openviking/openviking/service/resource_memory_link_service.py:284` `"archive_uri": commit_result.get("archive_uri"),`

**_build_resource_deletion_message**, body `sources/openviking/openviking/service/resource_memory_link_service.py:314`; implementation/call-through.

- `sources/openviking/openviking/service/resource_memory_link_service.py:306` `@staticmethod`
- `sources/openviking/openviking/service/resource_memory_link_service.py:307` `def _build_resource_deletion_message(`
- `sources/openviking/openviking/service/resource_memory_link_service.py:314` `affected = "\n".join(f"- {uri}" for uri in dict.fromkeys(memory_uris))`

**before_resource_delete**, body `sources/openviking/openviking/service/resource_memory_link_service.py:357`; implementation/call-through.

- `sources/openviking/openviking/service/resource_memory_link_service.py:349` `async def before_resource_delete(`
- `sources/openviking/openviking/service/resource_memory_link_service.py:401` `deleted.extend(cleanup_result.deleted_uris)`
- `sources/openviking/openviking/service/resource_memory_link_service.py:402` `if memory_uri in cleanup_result.deleted_uris:`

**unregister_after_delete**, body `sources/openviking/openviking/service/session_expiry.py:98`; implementation/call-through.

- `sources/openviking/openviking/service/session_expiry.py:97` `async def unregister_after_delete(self, identity: ItemIdentity) -> None:`
- `sources/openviking/openviking/service/session_expiry.py:98` `try:`

**_record_archive_metric**, body `sources/openviking/openviking/service/session_service.py:207`; implementation/call-through.

- `sources/openviking/openviking/service/session_service.py:204` `@staticmethod`
- `sources/openviking/openviking/service/session_service.py:205` `def _record_archive_metric(status: str) -> None:`
- `sources/openviking/openviking/service/session_service.py:210` `SessionLifecycleDataSource.record_archive(status=status)`
- `sources/openviking/openviking/service/session_service.py:213` `"Failed to record session archive metric status=%s",`

**delete**, body `sources/openviking/openviking/service/session_service.py:391`; implementation/call-through.

- `sources/openviking/openviking/service/session_service.py:382` `async def delete(self, session_id: str, ctx: RequestContext) -> bool:`
- `sources/openviking/openviking/service/session_service.py:396` `self._record_lifecycle_metric("delete", "error")`
- `sources/openviking/openviking/service/session_service.py:407` `self._record_lifecycle_metric("delete", "ok")`

**deleted_count**, body `sources/openviking/openviking/service/task_domain.py:86`; implementation/call-through.

- `sources/openviking/openviking/service/task_domain.py:84` `@property`
- `sources/openviking/openviking/service/task_domain.py:85` `def deleted_count(self) -> int:`
- `sources/openviking/openviking/service/task_domain.py:86` `return len(self.deleted)`

**clear**, body `sources/openviking/openviking/service/task_queue_middleware.py:267`; implementation/call-through.

- `sources/openviking/openviking/service/task_queue_middleware.py:266` `async def clear(self, ctx: ClearContext, call_next: ClearNext) -> None:`
- `sources/openviking/openviking/service/task_queue_middleware.py:267` `tracked = [`

**delete_tasks**, body `sources/openviking/openviking/service/task_store.py:191`; stub.

- `sources/openviking/openviking/service/task_store.py:184` `async def delete_tasks(`
- `sources/openviking/openviking/service/task_store.py:191` `) -> DeleteTasksResult: ...`

**delete_expired**, body `sources/openviking/openviking/service/task_store.py:195`; stub.

- `sources/openviking/openviking/service/task_store.py:193` `async def delete_expired(`
- `sources/openviking/openviking/service/task_store.py:195` `) -> DeleteExpiredResult: ...`

**delete_tasks**, body `sources/openviking/openviking/service/task_store.py:1156`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1147` `async def delete_tasks(`
- `sources/openviking/openviking/service/task_store.py:1156` `if task_id is not None:`

**_delete_owner_tasks**, body `sources/openviking/openviking/service/task_store.py:1189`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1182` `async def _delete_owner_tasks(`
- `sources/openviking/openviking/service/task_store.py:1208` `result = await self.delete_tasks(`
- `sources/openviking/openviking/service/task_store.py:1216` `deleted.extend(result.deleted)`

**_deletion_blocker**, body `sources/openviking/openviking/service/task_store.py:1232`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1226` `@staticmethod`
- `sources/openviking/openviking/service/task_store.py:1227` `def _deletion_blocker(`
- `sources/openviking/openviking/service/task_store.py:1232` `allowed = set(allow_active_types)`

**_delete_tasks_rejected**, body `sources/openviking/openviking/service/task_store.py:1257`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1255` `@staticmethod`
- `sources/openviking/openviking/service/task_store.py:1256` `def _delete_tasks_rejected(blockers: Collection[TaskDeletionBlocker]) -> DeleteTasksResult:`
- `sources/openviking/openviking/service/task_store.py:1257` `ordered = tuple(blockers)`

**_delete_task_unlocked**, body `sources/openviking/openviking/service/task_store.py:1283`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1278` `async def _delete_task_unlocked(`
- `sources/openviking/openviking/service/task_store.py:1283` `account_id, user_id, task_id = owner`

**delete_expired**, body `sources/openviking/openviking/service/task_store.py:1297`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1294` `async def delete_expired(`
- `sources/openviking/openviking/service/task_store.py:1297` `if limit <= 0:`

**_rm**, body `sources/openviking/openviking/service/task_store.py:1566`; implementation/call-through.

- `sources/openviking/openviking/service/task_store.py:1565` `async def _rm(self, path: str) -> None:`
- `sources/openviking/openviking/service/task_store.py:1566` `await run_to_completion(lambda: self._agfs.rm(path, force=True, auto_pathlock=False))`

**delete_tasks**, body `sources/openviking/openviking/service/task_store_mysql.py:758`; implementation/call-through.

- `sources/openviking/openviking/service/task_store_mysql.py:750` `async def delete_tasks(`
- `sources/openviking/openviking/service/task_store_mysql.py:758` `if not account_id:`

**delete_expired**, body `sources/openviking/openviking/service/task_store_mysql.py:843`; implementation/call-through.

- `sources/openviking/openviking/service/task_store_mysql.py:840` `async def delete_expired(`
- `sources/openviking/openviking/service/task_store_mysql.py:887` `await session.delete(row)`

**delete_expired**, body `sources/openviking/openviking/service/task_tracker.py:379`; implementation/call-through.

- `sources/openviking/openviking/service/task_tracker.py:378` `async def delete_expired(self) -> DeleteExpiredResult:`
- `sources/openviking/openviking/service/task_tracker.py:379` `result = await self._store.delete_expired(`
- `sources/openviking/openviking/service/task_tracker.py:384` `for deleted in result.deleted:`

**delete_tasks**, body `sources/openviking/openviking/service/task_tracker.py:1341`; implementation/call-through.

- `sources/openviking/openviking/service/task_tracker.py:1334` `async def delete_tasks(`
- `sources/openviking/openviking/service/task_tracker.py:1344` `delete_result = await self._store.delete_tasks(`
- `sources/openviking/openviking/service/task_tracker.py:1351` `for task in delete_result.deleted:`
- `sources/openviking/openviking/service/task_tracker.py:1352` `self._work_index.forget_task(`
- `sources/openviking/openviking/service/task_tracker.py:1357` `return len(delete_result.deleted)`

**forget_task**, body `sources/openviking/openviking/service/task_work_index.py:263`; implementation/call-through.

- `sources/openviking/openviking/service/task_work_index.py:256` `def forget_task(`
- `sources/openviking/openviking/service/task_work_index.py:263` `with self._lock:`

**archive_index_from_uri**, body `sources/openviking/openviking/session/archive_store.py:119`; implementation/call-through.

- `sources/openviking/openviking/session/archive_store.py:116` `@staticmethod`
- `sources/openviking/openviking/session/archive_store.py:117` `def archive_index_from_uri(archive_uri: str) -> int:`
- `sources/openviking/openviking/session/archive_store.py:119` `match = re.search(r"archive_(\d+)$", archive_uri.rstrip("/"))`
- `sources/openviking/openviking/session/archive_store.py:121` `raise ValueError(f"Invalid archive URI: {archive_uri}")`

**covered_archive_ids**, body `sources/openviking/openviking/session/archive_store.py:139`; implementation/call-through.

- `sources/openviking/openviking/session/archive_store.py:136` `@staticmethod`
- `sources/openviking/openviking/session/archive_store.py:137` `def covered_archive_ids(states: List[ArchiveState]) -> set[str]:`
- `sources/openviking/openviking/session/archive_store.py:139` `existing = {state.archive_id: state for state in states}`
- `sources/openviking/openviking/session/archive_store.py:153` `# A pending archive still has a live Phase 2 owner and is never`
- `sources/openviking/openviking/session/archive_store.py:157` `covered.add(candidate.archive_id)`
- `sources/openviking/openviking/session/archive_store.py:158` `explicit = state.done.get("covered_failed_archives", [])`
- `sources/openviking/openviking/session/archive_store.py:161` `archive_id`
- `sources/openviking/openviking/session/archive_store.py:162` `for archive_id in explicit`
- `sources/openviking/openviking/session/archive_store.py:163` `if isinstance(archive_id, str)`
- `sources/openviking/openviking/session/archive_store.py:164` `and archive_id in existing`
- `sources/openviking/openviking/session/archive_store.py:165` `and existing[archive_id].index <= state.index`
- `sources/openviking/openviking/session/archive_store.py:166` `and existing[archive_id].state == "failed"`

**is_context_reset_archive**, body `sources/openviking/openviking/session/archive_store.py:384`; implementation/call-through.

- `sources/openviking/openviking/session/archive_store.py:382` `async def is_context_reset_archive(self, archive_uri: str) -> bool:`
- `sources/openviking/openviking/session/archive_store.py:384` `return (await self.read_done(archive_uri)).get("context_reset") is True`

**_normalize_delete_ids**, body `sources/openviking/openviking/session/memory/extract_loop.py:1035`; implementation/call-through.

- `sources/openviking/openviking/session/memory/extract_loop.py:1034` `def _normalize_delete_ids(self, raw_delete_ids: List[Any]) -> List[DeleteId]:`
- `sources/openviking/openviking/session/memory/extract_loop.py:1035` `delete_ids: List[DeleteId] = []`

**_same_batch_delete_conflict_key**, body `sources/openviking/openviking/session/memory/memory_updater.py:768`; implementation/call-through.

- `sources/openviking/openviking/session/memory/memory_updater.py:760` `def _same_batch_delete_conflict_key(uri: str) -> str:`
- `sources/openviking/openviking/session/memory/memory_updater.py:768` `return str(uri or "").rstrip("/").casefold()`

**get_source_archive_suffix**, body `sources/openviking/openviking/session/memory/memory_updater.py:417`; implementation/call-through.

- `sources/openviking/openviking/session/memory/memory_updater.py:411` `def get_source_archive_suffix(self) -> str:`
- `sources/openviking/openviking/session/memory/memory_updater.py:417` `if not self.source_archive_uri:`
- `sources/openviking/openviking/session/memory/memory_updater.py:419` `digest = hashlib.sha256(self.source_archive_uri.encode("utf-8")).hexdigest()[:16]`

**add_deleted**, body `sources/openviking/openviking/session/memory/memory_updater.py:743`; implementation/call-through.

- `sources/openviking/openviking/session/memory/memory_updater.py:742` `def add_deleted(self, uri: str) -> None:`
- `sources/openviking/openviking/session/memory/memory_updater.py:743` `self.deleted_uris.append(uri)`

**_inherit_deleted_link_relations**, body `sources/openviking/openviking/session/memory/memory_updater.py:1601`; implementation/call-through.

- `sources/openviking/openviking/session/memory/memory_updater.py:1594` `async def _inherit_deleted_link_relations(`
- `sources/openviking/openviking/session/memory/memory_updater.py:1601` `uri_remap = dict(getattr(operations, "delete_replacements", {}) or {})`

**_apply_delete**, body `sources/openviking/openviking/session/memory/memory_updater.py:1744`; implementation/call-through.

- `sources/openviking/openviking/session/memory/memory_updater.py:1737` `async def _apply_delete(`
- `sources/openviking/openviking/session/memory/memory_updater.py:1746` `# Delete from VikingFS`

**_delete_complete_lines**, body `sources/openviking/openviking/session/memory/merge_op/patch_handler.py:231`; implementation/call-through.

- `sources/openviking/openviking/session/memory/merge_op/patch_handler.py:229` `def _delete_complete_lines(content: str, delete_content: str) -> str:`
- `sources/openviking/openviking/session/memory/merge_op/patch_handler.py:231` `start = content.index(delete_content)`

**memory_file_to_delete_patch**, body `sources/openviking/openviking/session/memory/streaming_memory_updater.py:1136`; implementation/call-through.

- `sources/openviking/openviking/session/memory/streaming_memory_updater.py:1124` `def memory_file_to_delete_patch(`
- `sources/openviking/openviking/session/memory/streaming_memory_updater.py:1136` `after_file = MemoryFile(`

**has_archive_work**, body `sources/openviking/openviking/session/retention.py:93`; implementation/call-through.

- `sources/openviking/openviking/session/retention.py:91` `@property`
- `sources/openviking/openviking/session/retention.py:92` `def has_archive_work(self) -> bool:`
- `sources/openviking/openviking/session/retention.py:93` `return bool(self.archive_messages)`

**_merge_archive_meta**, body `sources/openviking/openviking/session/session.py:1054`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:1047` `async def _merge_archive_meta(`
- `sources/openviking/openviking/session/session.py:1059` `async with self._archive_meta_merge_lock:`
- `sources/openviking/openviking/session/session.py:1062` `content = await self._viking_fs.read_file(f"{archive_uri}/.meta.json", ctx=self.ctx)`
- `sources/openviking/openviking/session/session.py:1070` `uri=f"{archive_uri}/.meta.json",`

**_archive_file_exists**, body `sources/openviking/openviking/session/session.py:1156`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:1155` `async def _archive_file_exists(self, archive_uri: str, file_name: str) -> bool:`
- `sources/openviking/openviking/session/session.py:1157` `return await self._viking_fs.exists(f"{archive_uri}/{file_name}", ctx=self.ctx)`

**_append_context_reset_archive**, body `sources/openviking/openviking/session/session.py:1811`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:1804` `async def _append_context_reset_archive(self) -> None:`
- `sources/openviking/openviking/session/session.py:1811` `newest = f"{self._session_uri}/history/archive_{self._compression.compression_index:03d}"`
- `sources/openviking/openviking/session/session.py:1814` `and await self._archives.is_context_reset_archive(newest)`
- `sources/openviking/openviking/session/session.py:1818` `archive_uri = (`
- `sources/openviking/openviking/session/session.py:1819` `f"{self._session_uri}/history/archive_{self._compression.compression_index:03d}"`
- `sources/openviking/openviking/session/session.py:1823` `f"{archive_uri}/.done",`
- `sources/openviking/openviking/session/session.py:1829` `# block the next Phase 2 forever; no queue owns this archive.`
- `sources/openviking/openviking/session/session.py:1830` `await self._write_failed_marker(archive_uri, stage="context_reset", error=str(exc))`

**get_session_archive**, body `sources/openviking/openviking/session/session.py:2753`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:2750` `async def get_session_archive(self, archive_id: str) -> Dict[str, Any]:`
- `sources/openviking/openviking/session/session.py:2753` `for archive in await self._archives.completed_refs():`
- `sources/openviking/openviking/session/session.py:2754` `if archive["archive_id"] != archive_id:`
- `sources/openviking/openviking/session/session.py:2757` `overview = await self._archives.read_overview(archive["archive_uri"])`
- `sources/openviking/openviking/session/session.py:2758` `if not overview and archive.get("enable_working_memory") is not False:`
- `sources/openviking/openviking/session/session.py:2762` `await self._archives.read_abstract(archive["archive_uri"], overview)`
- `sources/openviking/openviking/session/session.py:2767` `"archive_id": archive_id,`
- `sources/openviking/openviking/session/session.py:2771` `m.to_dict() for m in await self._archives.read_messages(archive["archive_uri"])`
- `sources/openviking/openviking/session/session.py:2775` `raise NotFoundError(archive_id, "session archive")`

**_can_run_archive**, body `sources/openviking/openviking/session/session.py:2876`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:2874` `async def _can_run_archive(self, archive_index: int) -> bool:`
- `sources/openviking/openviking/session/session.py:2876` `if archive_index <= 1 or not self._viking_fs:`
- `sources/openviking/openviking/session/session.py:2879` `predecessor_uri = f"{self._session_uri}/history/archive_{archive_index - 1:03d}"`
- `sources/openviking/openviking/session/session.py:2882` `if await self._archives.terminal_state(predecessor_uri) != "pending":`
- `sources/openviking/openviking/session/session.py:2902` `# A ready archive can still be between Task creation and QueueFS`
- `sources/openviking/openviking/session/session.py:2914` `"Predecessor archive is still being published; requeueing: %s",`
- `sources/openviking/openviking/session/session.py:2937` `"Skipped orphaned Session archive without QueueFS work: %s",`

**_prepare_phase2_archive_messages**, body `sources/openviking/openviking/session/session.py:2950`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:2944` `async def _prepare_phase2_archive_messages(`
- `sources/openviking/openviking/session/session.py:2950` `current_archive_id = archive_uri.rstrip("/").split("/")[-1]`
- `sources/openviking/openviking/session/session.py:2952` `current_meta = await self._archives.read_meta(archive_uri)`
- `sources/openviking/openviking/session/session.py:2953` `self._archives.merge_completed_memory_steps(`
- `sources/openviking/openviking/session/session.py:2959` `current_archive_id,`
- `sources/openviking/openviking/session/session.py:2960` `current_archive_id,`

**_generate_archive_summary_with_batching**, body `sources/openviking/openviking/session/session.py:3071`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:3064` `async def _generate_archive_summary_with_batching(`
- `sources/openviking/openviking/session/session.py:3074` `return await self._generate_archive_summary_async(`
- `sources/openviking/openviking/session/session.py:3076` `latest_archive_overview=latest_archive_overview,`
- `sources/openviking/openviking/session/session.py:3081` `return await self._generate_archive_summary_async(`
- `sources/openviking/openviking/session/session.py:3083` `latest_archive_overview=latest_archive_overview,`
- `sources/openviking/openviking/session/session.py:3093` `current_overview = latest_archive_overview`
- `sources/openviking/openviking/session/session.py:3097` `generated = await self._generate_archive_summary_async(`
- `sources/openviking/openviking/session/session.py:3099` `latest_archive_overview=current_overview,`
- `sources/openviking/openviking/session/session.py:3103` `if isinstance(generated, _ArchiveSummaryResult)`

**_generate_archive_summary_async**, body `sources/openviking/openviking/session/session.py:3126`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:3108` `async def _generate_archive_summary_async(`
- `sources/openviking/openviking/session/session.py:3127` `f"_generate_archive_summary_async called "`
- `sources/openviking/openviking/session/session.py:3128` `f"messages={len(messages)} prior_wm={len(latest_archive_overview)}B"`
- `sources/openviking/openviking/session/session.py:3133` `raise ValueError("Cannot generate checkpoints without archive messages")`
- `sources/openviking/openviking/session/session.py:3171` `_is_wm_v2 = latest_archive_overview and any(`
- `sources/openviking/openviking/session/session.py:3172` `f"## {s}" in latest_archive_overview for s in WM_SEVEN_SECTIONS`
- `sources/openviking/openviking/session/session.py:3176` `if not latest_archive_overview or not _is_wm_v2:`
- `sources/openviking/openviking/session/session.py:3178` `f"branch=CREATE (prior={'legacy' if latest_archive_overview else 'none'} "`
- `sources/openviking/openviking/session/session.py:3179` `f"{len(latest_archive_overview or '')}B)"`
- `sources/openviking/openviking/session/session.py:3185` `"latest_archive_overview": latest_archive_overview or "",`
- `sources/openviking/openviking/session/session.py:3215` `return _ArchiveSummaryResult(`
- `sources/openviking/openviking/session/session.py:3225` `wm.wm_debug(f"branch=UPDATE (prior WM={len(latest_archive_overview)}B)")`
- `sources/openviking/openviking/session/session.py:3226` `reminders = wm.build_wm_section_reminders(latest_archive_overview)`
- `sources/openviking/openviking/session/session.py:3233` `"latest_archive_overview": latest_archive_overview,`
- `sources/openviking/openviking/session/session.py:3263` `formatted, messages, latest_archive_overview, output_language`
- `sources/openviking/openviking/session/session.py:3356` `return wm.merge_wm_sections(latest_archive_overview, salvaged)`
- `sources/openviking/openviking/session/session.py:3364` `formatted, messages, latest_archive_overview, output_language`
- `sources/openviking/openviking/session/session.py:3371` `overview = wm.merge_wm_sections(latest_archive_overview, ops)`
- `sources/openviking/openviking/session/session.py:3373` `return _ArchiveSummaryResult(`

**_run_archive_summary**, body `sources/openviking/openviking/session/session.py:2181`; implementation/call-through.

- `sources/openviking/openviking/session/session.py:2180` `async def _run_archive_summary() -> None:`
- `sources/openviking/openviking/session/session.py:2188` `"latest_archive_overview": latest_archive_overview,`
- `sources/openviking/openviking/session/session.py:2197` `generated = await self._generate_archive_summary_async(`
- `sources/openviking/openviking/session/session.py:2202` `generated = await self._generate_archive_summary_with_batching(`
- `sources/openviking/openviking/session/session.py:2209` `if isinstance(generated, _ArchiveSummaryResult)`
- `sources/openviking/openviking/session/session.py:2210` `else _ArchiveSummaryResult(overview=str(generated or ""))`
- `sources/openviking/openviking/session/session.py:2224` `uri=f"{archive_uri}/.abstract.md",`
- `sources/openviking/openviking/session/session.py:2227` `archive_uri,`
- `sources/openviking/openviking/session/session.py:2232` `"trigger": "archive_summary",`
- `sources/openviking/openviking/session/session.py:2239` `uri=f"{archive_uri}/.overview.md",`
- `sources/openviking/openviking/session/session.py:2242` `archive_uri,`
- `sources/openviking/openviking/session/session.py:2247` `"trigger": "archive_summary",`
- `sources/openviking/openviking/session/session.py:2253` `await self._merge_archive_meta(`
- `sources/openviking/openviking/session/session.py:2254` `archive_uri,`

**_apply_trajectory_source_archive_uri**, body `sources/openviking/openviking/session/train/components/trajectory_analyzer.py:431`; implementation/call-through.

- `sources/openviking/openviking/session/train/components/trajectory_analyzer.py:427` `def _apply_trajectory_source_archive_uri(`
- `sources/openviking/openviking/session/train/components/trajectory_analyzer.py:431` `source_archive_uri = str(source_archive_uri or "").rstrip("/")`
- `sources/openviking/openviking/session/train/components/trajectory_analyzer.py:432` `if not source_archive_uri:`
- `sources/openviking/openviking/session/train/components/trajectory_analyzer.py:440` `fields["source_archive_uri"] = source_archive_uri`

**_write_ovpack_archive**, body `sources/openviking/openviking/storage/ovpack/operations.py:431`; implementation/call-through.

- `sources/openviking/openviking/storage/ovpack/operations.py:418` `async def _write_ovpack_archive(`
- `sources/openviking/openviking/storage/ovpack/operations.py:431` `ensure_dir_exists(to)`

**for_delete**, body `sources/openviking/openviking/storage/queuefs/embedding_msg.py:287`; implementation/call-through.

- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:279` `@classmethod`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:280` `def for_delete(`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:288` `action=IndexAction.DELETE,`

**clear**, body `sources/openviking/openviking/storage/queuefs/named_queue.py:396`; implementation/call-through.

- `sources/openviking/openviking/storage/queuefs/named_queue.py:394` `async def clear(self) -> bool:`
- `sources/openviking/openviking/storage/queuefs/named_queue.py:396` `await self._ensure_initialized()`

**clear**, body `sources/openviking/openviking/storage/queuefs/queue_manager.py:546`; implementation/call-through.

- `sources/openviking/openviking/storage/queuefs/queue_manager.py:544` `async def clear(self, queue_name: str) -> bool:`
- `sources/openviking/openviking/storage/queuefs/queue_manager.py:546` `return await self.get_queue(queue_name).clear()`

**clear**, body `sources/openviking/openviking/storage/queuefs/queue_middleware.py:96`; implementation/call-through.

- `sources/openviking/openviking/storage/queuefs/queue_middleware.py:95` `async def clear(self, ctx: ClearContext, call_next: ClearNext) -> None:`
- `sources/openviking/openviking/storage/queuefs/queue_middleware.py:96` `await call_next(ctx)`

**delete_path**, body `sources/openviking/openviking/storage/resource_target.py:61`; implementation/call-through.

- `sources/openviking/openviking/storage/resource_target.py:59` `async def delete_path(self, rel_path: str, *, is_dir: bool) -> None:`
- `sources/openviking/openviking/storage/resource_target.py:61` `await self._viking_fs.remove_files(`

**delete_vector_records**, body `sources/openviking/openviking/storage/vector_migration.py:170`; implementation/call-through.

- `sources/openviking/openviking/storage/vector_migration.py:162` `async def delete_vector_records(`
- `sources/openviking/openviking/storage/vector_migration.py:174` `or not hasattr(vector_store, "delete")`
- `sources/openviking/openviking/storage/vector_migration.py:197` `result.deleted = int(await vector_store.delete(ids, ctx=ctx) or 0)`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/collection.py:175`; stub.

- `sources/openviking/openviking/storage/vectordb/collection/collection.py:173` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:174` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:175` `raise NotImplementedError`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/collection.py:179`; stub.

- `sources/openviking/openviking/storage/vectordb/collection/collection.py:177` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:178` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:179` `raise NotImplementedError`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/collection.py:684`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/collection.py:677` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:686` `return self.__collection.delete_data(primary_keys)`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/collection.py:693`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/collection.py:688` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/collection.py:695` `return self.__collection.delete_all_data()`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:355`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:354` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:355` `url = self.url_prefix + "api/vikingdb/data/delete"`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:371`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:370` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:371` `url = self.url_prefix + "api/vikingdb/data/delete"`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:941`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:940` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:951` `delta_list = self.store_mgr.delete_data(labels_list, need_record_delta)`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:955` `index.delete_data(delta_list)`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:974`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:959` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:974` `with self._index_mutation_barrier.exclusive_mutation():`

**_delete_expire_delta_record**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1031`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1030` `def _delete_expire_delta_record(self):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1037` `self.store_mgr.delete_delta_data_before_ts(oldest_version)`

**_delete_scheduler_job**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1064`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1063` `def _delete_scheduler_job(self):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1066` `self.scheduler.remove_job(self.ttl_cleanup_job_id)`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1075` `self.scheduler.remove_job(self.index_manage_job_id)`

**delete_from_index**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:955`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:954` `def delete_from_index(name, index):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:955` `index.delete_data(delta_list)`

**delete_from_index**, body `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1048`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1047` `def delete_from_index(name, index):`
- `sources/openviking/openviking/storage/vectordb/collection/local_collection.py:1048` `index.delete_data(delta_list)`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:235`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:234` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:235` `path = "/api/vikingdb/data/delete"`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:244`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:243` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_collection.py:244` `path = "/api/vikingdb/data/delete"`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:340`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:339` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:340` `path = "/api/vikingdb/data/delete"`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:348`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:347` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_api_key_collection.py:348` `path = "/api/vikingdb/data/delete"`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:450`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:449` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:450` `path = "/api/vikingdb/data/delete"`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:459`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:458` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_collection.py:459` `path = "/api/vikingdb/data/delete"`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:472`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:471` `def delete_data(self, data_list: list[DeleteDataRequest]) -> int:`
- `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:472` `return int(`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:599`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:598` `def delete_data(self, keys: list[str]) -> int:`
- `sources/openviking/openviking/storage/vectordb/engine/_python_api.py:599` `return int(self._backend._store_delete_data(self._handle, keys))`

**delete**, body `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1105`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1104` `def delete(self, records: Iterable[DeltaRecord]) -> None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1111` `self._invalidate()`

**_invalidate**, body `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1114`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1113` `def _invalidate(self) -> None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1114` `self._records_generation += 1`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/index/index.py:88`; stub.

- `sources/openviking/openviking/storage/vectordb/index/index.py:69` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/index/index.py:70` `def delete_data(self, delta_list: List[DeltaRecord]):`
- `sources/openviking/openviking/storage/vectordb/index/index.py:88` `raise NotImplementedError`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/index/index.py:336`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/index/index.py:324` `def delete_data(self, delta_list: List[DeltaRecord]):`
- `sources/openviking/openviking/storage/vectordb/index/index.py:338` `self.__index.delete_data(delta_list)`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/index/local_index.py:244`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/index/local_index.py:243` `def delete_data(self, delta_list: List[DeltaRecord]):`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:247` `del_req_list = [engine.DeleteDataRequest() for _ in range(len(delta_list))]`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:251` `self.index_engine.delete_data(del_req_list)`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/index/local_index.py:541`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/index/local_index.py:540` `def delete_data(self, delta_list: List[DeltaRecord]):`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:544` `self.engine_proxy.delete_data(self._convert_delta_list_for_index(delta_list))`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:545` `self.dense_search.delete(delta_list)`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:548` `self.engine_proxy.delete_data(self._convert_delta_list_for_index(delta_list))`

**delete_project**, body `sources/openviking/openviking/storage/vectordb/project/project_group.py:248`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/project/project_group.py:241` `def delete_project(self, project_name: str):`
- `sources/openviking/openviking/storage/vectordb/project/project_group.py:248` `project = self.projects.remove(project_name)`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:276`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:273` `@data_router.post("/delete", response_model=ApiResponse)`
- `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:274` `async def delete_data(request: DataDeleteRequest, req: Request):`
- `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:283` `collection.delete_all_data()`
- `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:286` `collection.delete_data(primary_keys=primary_keys)`

**delete**, body `sources/openviking/openviking/storage/vectordb/store/file_store.py:153`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/file_store.py:143` `def delete(self, key: str) -> bool:`
- `sources/openviking/openviking/storage/vectordb/store/file_store.py:156` `path.unlink()`

**delete**, body `sources/openviking/openviking/storage/vectordb/store/local_store.py:92`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/local_store.py:85` `def delete(self, keys: List[str], table_name: str):`
- `sources/openviking/openviking/storage/vectordb/store/local_store.py:93` `self._check_mutation_status(self.storage_engine.delete_data(keys), "delete_data")`

**clear**, body `sources/openviking/openviking/storage/vectordb/store/local_store.py:97`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/local_store.py:95` `def clear(self):`
- `sources/openviking/openviking/storage/vectordb/store/local_store.py:97` `self._check_mutation_status(self.storage_engine.clear_data(), "clear_data")`

**delete**, body `sources/openviking/openviking/storage/vectordb/store/store.py:56`; stub.

- `sources/openviking/openviking/storage/vectordb/store/store.py:46` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/store/store.py:47` `def delete(self, key):`
- `sources/openviking/openviking/storage/vectordb/store/store.py:56` `raise NotImplementedError`

**delete**, body `sources/openviking/openviking/storage/vectordb/store/store.py:163`; stub.

- `sources/openviking/openviking/storage/vectordb/store/store.py:152` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/store/store.py:153` `def delete(self, keys: List[str], table_name: str):`
- `sources/openviking/openviking/storage/vectordb/store/store.py:163` `pass`

**clear**, body `sources/openviking/openviking/storage/vectordb/store/store.py:175`; stub.

- `sources/openviking/openviking/storage/vectordb/store/store.py:165` `@abstractmethod`
- `sources/openviking/openviking/storage/vectordb/store/store.py:166` `def clear(self):`
- `sources/openviking/openviking/storage/vectordb/store/store.py:175` `pass`

**delete_data**, body `sources/openviking/openviking/storage/vectordb/store/store_manager.py:142`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:130` `def delete_data(`
- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:157` `delta_list = [DeltaRecord(type=DeltaRecord.Type.DELETE) for _ in range(len(label_list))]`

**clear**, body `sources/openviking/openviking/storage/vectordb/store/store_manager.py:250`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:248` `def clear(self):`
- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:250` `self.storage.clear()`

**delete_delta_data_before_ts**, body `sources/openviking/openviking/storage/vectordb/store/store_manager.py:271`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:260` `def delete_delta_data_before_ts(self, ns_ts: int) -> int:`
- `sources/openviking/openviking/storage/vectordb/store/store_manager.py:278` `self.storage.delete(delta_keys, StoreManager.DeltaTable)`

**clear**, body `sources/openviking/openviking/storage/vectordb/utils/dict_utils.py:62`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb/utils/dict_utils.py:60` `def clear(self):`
- `sources/openviking/openviking/storage/vectordb/utils/dict_utils.py:62` `with self._lock:`

**delete**, body `sources/openviking/openviking/storage/vectordb_adapters/base.py:616`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/base.py:609` `def delete(`
- `sources/openviking/openviking/storage/vectordb_adapters/base.py:620` `coll.delete_data(ids[start : start + batch_size])`
- `sources/openviking/openviking/storage/vectordb_adapters/base.py:645` `coll.delete_data(json.loads(batch))`

**clear**, body `sources/openviking/openviking/storage/vectordb_adapters/base.py:744`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/base.py:743` `def clear(self) -> bool:`
- `sources/openviking/openviking/storage/vectordb_adapters/base.py:744` `self.get_collection().delete_all_data()`

**delete_data**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:80`; stub.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:78` `def delete_data(self, delta_list):`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:80` `pass`

**_delete_index_meta**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:286`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:285` `def _delete_index_meta(self, index_name: str):`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:291` `f"DELETE FROM {idx_table} WHERE index_name = %s",`

**_delete_index_meta_and_scalar_indexes**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:926`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:921` `def _delete_index_meta_and_scalar_indexes(`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:940` `f"DELETE FROM {idx_table} WHERE index_name = %s",`

**delete_data**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1567`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1566` `def delete_data(self, primary_keys: List[Any]):`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1572` `f'DELETE FROM "{self._name}" WHERE id IN ({placeholders})',`

**delete_all_data**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1577`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1576` `def delete_all_data(self):`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss/collection.py:1577` `self._execute(f'TRUNCATE TABLE "{self._name}"')`

**delete**, body `sources/openviking/openviking/storage/vectordb_adapters/opengauss_adapter.py:278`; implementation/call-through.

- `sources/openviking/openviking/storage/vectordb_adapters/opengauss_adapter.py:271` `def delete(`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss_adapter.py:279` `return super().delete(ids=ids, filter=filter)`
- `sources/openviking/openviking/storage/vectordb_adapters/opengauss_adapter.py:327` `f'DELETE FROM "{self._collection_name}" '`

**set_deletion_guard**, body `sources/openviking/openviking/storage/viking_fs/_access.py:91`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_access.py:90` `def set_deletion_guard(self, guard: Optional[Callable[[str, str], bool]]) -> None:`
- `sources/openviking/openviking/storage/viking_fs/_access.py:91` `self._deletion_guard = guard`

**delete_acl**, body `sources/openviking/openviking/storage/viking_fs/_access.py:405`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_access.py:404` `async def delete_acl(self, uri: str, ctx: Optional[RequestContext] = None) -> Dict[str, Any]:`
- `sources/openviking/openviking/storage/viking_fs/_access.py:405` `return await self.set_acl(uri, [], acl_mode=AclMode.INHERIT, ctx=ctx)`

**_ensure_identity_not_deleting**, body `sources/openviking/openviking/storage/viking_fs/_access.py:408`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_access.py:407` `async def _ensure_identity_not_deleting(self, ctx: RequestContext) -> None:`
- `sources/openviking/openviking/storage/viking_fs/_access.py:408` `guard = getattr(self, "_deletion_guard", None)`

**_ensure_supported_delete_namespace**, body `sources/openviking/openviking/storage/viking_fs/_access.py:415`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_access.py:414` `def _ensure_supported_delete_namespace(self, normalized_uri: str) -> None:`
- `sources/openviking/openviking/storage/viking_fs/_access.py:418` `"Deleting viking:// is not supported; use a concrete scope instead.",`
- `sources/openviking/openviking/storage/viking_fs/_access.py:423` `"Deleting viking://user is not supported; use viking://~/... or an "`
- `sources/openviking/openviking/storage/viking_fs/_access.py:429` `"Deleting viking://agent root is not supported; use a concrete "`

**rm**, body `sources/openviking/openviking/storage/viking_fs/_ops.py:250`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_ops.py:216` `async def rm(`
- `sources/openviking/openviking/storage/viking_fs/_ops.py:250` `from openviking.storage.errors import LockAcquisitionError, ResourceBusyError`

**delete_temp**, body `sources/openviking/openviking/storage/viking_fs/_ops.py:2594`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_ops.py:2587` `async def delete_temp(`
- `sources/openviking/openviking/storage/viking_fs/_ops.py:2594` `await self._ensure_access(temp_uri, ctx, action=AclAction.MANAGE)`

**_estimate_deleted_count**, body `sources/openviking/openviking/storage/viking_fs/_ops.py:260`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_ops.py:258` `async def _estimate_deleted_count(target_path: str, real_ctx: RequestContext) -> int:`
- `sources/openviking/openviking/storage/viking_fs/_ops.py:260` `vector_store = self._get_vector_store()`

**delete_gitignore**, body `sources/openviking/openviking/storage/viking_fs/_snapshot.py:183`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_snapshot.py:181` `async def delete_gitignore(self, ctx: Optional[RequestContext] = None) -> None:`
- `sources/openviking/openviking/storage/viking_fs/_snapshot.py:183` `real_ctx = self._ctx_or_default(ctx)`

**_delete_from_vector_store**, body `sources/openviking/openviking/storage/viking_fs/_vector.py:25`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_fs/_vector.py:18` `async def _delete_from_vector_store(`
- `sources/openviking/openviking/storage/viking_fs/_vector.py:31` `await vector_store.delete_uris(real_ctx, uris)`
- `sources/openviking/openviking/storage/viking_fs/_vector.py:35` `logger.warning(f"[VikingFS] Failed to delete from vector store: {e}")`

**strict_delete**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:776`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:773` `@_backend_operation`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:774` `async def strict_delete(self, ids: List[str]) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:782` `return int(await self._async_adapter.call("delete", ids=ids) or 0)`

**delete**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:817`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:815` `@_backend_operation`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:816` `async def delete(self, ids: List[str]) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:825` `return await self._async_adapter.call("delete", ids=ids)`

**delete_by_filter**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:833`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:830` `@_backend_operation`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:831` `async def delete_by_filter(self, filter: FilterExpr) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:834` `return await self._async_adapter.call("delete", filter=filter)`

**clear**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:1050`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1048` `@_backend_operation`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1049` `async def clear(self) -> bool:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1052` `return await self.delete_by_filter(Eq("account_id", self._bound_account_id)) > 0`

**delete**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:1497`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1496` `async def delete(self, ids: List[str], *, ctx: RequestContext) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1498` `return await backend.delete(ids)`

**strict_delete**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:1502`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1500` `async def strict_delete(self, ids: List[str], *, ctx: RequestContext) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1502` `return await (await self._get_backend_for_context(ctx)).strict_delete(ids)`

**_strict_transfer_delete**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:1834`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1833` `async def _strict_transfer_delete(self, ctx: RequestContext, ids: List[str]) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1834` `backend = await self._get_backend_for_context(ctx)`

**clear**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:1888`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1887` `async def clear(self, *, ctx: Optional[RequestContext] = None) -> bool:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:1888` `if ctx:`

**delete_account_data**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:2536`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2534` `async def delete_account_data(self, account_id: str, *, ctx: RequestContext) -> int:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2538` `return await root_backend.delete_by_filter(Eq("account_id", account_id))`

**delete_user_data**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:2548`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2540` `async def delete_user_data(`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2550` `return await root_backend.delete_by_filter(`

**delete_uris**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:2555`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2554` `async def delete_uris(self, ctx: RequestContext, uris: List[str]) -> None:`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2562` `await backend.delete_by_filter(And(conds))`

**_delete_vector_transfer_ids**, body `sources/openviking/openviking/storage/viking_vector_index_backend.py:2709`; implementation/call-through.

- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2702` `async def _delete_vector_transfer_ids(`
- `sources/openviking/openviking/storage/viking_vector_index_backend.py:2709` `deleted = 0`

**delete**, body `sources/openviking/openviking/storage/vikingdb_manager.py:311`; implementation/call-through.

- `sources/openviking/openviking/storage/vikingdb_manager.py:310` `async def delete(self, ids: List[str]) -> int:`
- `sources/openviking/openviking/storage/vikingdb_manager.py:311` `return await self._manager.delete(ids, ctx=self._ctx)`

**clear**, body `sources/openviking/openviking/storage/vikingdb_manager.py:405`; implementation/call-through.

- `sources/openviking/openviking/storage/vikingdb_manager.py:404` `async def clear(self) -> bool:`
- `sources/openviking/openviking/storage/vikingdb_manager.py:405` `return await self._manager.clear(ctx=self._ctx)`

**delete_account_data**, body `sources/openviking/openviking/storage/vikingdb_manager.py:514`; implementation/call-through.

- `sources/openviking/openviking/storage/vikingdb_manager.py:513` `async def delete_account_data(self, account_id: str) -> int:`
- `sources/openviking/openviking/storage/vikingdb_manager.py:514` `return await self._manager.delete_account_data(account_id, ctx=self._ctx)`

**delete_uris**, body `sources/openviking/openviking/storage/vikingdb_manager.py:517`; implementation/call-through.

- `sources/openviking/openviking/storage/vikingdb_manager.py:516` `async def delete_uris(self, uris: List[str]) -> None:`
- `sources/openviking/openviking/storage/vikingdb_manager.py:517` `return await self._manager.delete_uris(self._ctx, uris)`

**_delete_removed_resource_vectors**, body `sources/openviking/openviking/utils/resource_processor.py:1674`; implementation/call-through.

- `sources/openviking/openviking/utils/resource_processor.py:1667` `async def _delete_removed_resource_vectors(`
- `sources/openviking/openviking/utils/resource_processor.py:1683` `await self.vikingdb.delete(ids, ctx=ctx)`
- `sources/openviking/openviking/utils/resource_processor.py:1699` `await self.vikingdb.delete(ids, ctx=ctx)`

**delete_data**, body `sources/openviking/openviking_gateway/app.py:307`; implementation/call-through.

- `sources/openviking/openviking_gateway/app.py:305` `async def delete_data(app, account_id, user_id=None):`
- `sources/openviking/openviking_gateway/app.py:307` `if user_id is None:`

**observe_archive**, body `sources/openviking/openviking_gateway/archives.py:17`; implementation/call-through.

- `sources/openviking/openviking_gateway/archives.py:15` `async def observe_archive(viking, archive, token):`
- `sources/openviking/openviking_gateway/archives.py:18` `if archive["status"] in TERMINAL or archive.get("next_check", 0) > now:`
- `sources/openviking/openviking_gateway/archives.py:19` `return archive`
- `sources/openviking/openviking_gateway/archives.py:22` `viking.archive_state(token, archive["archive_uri"]),`
- `sources/openviking/openviking_gateway/archives.py:27` `if status not in TERMINAL and now - archive["created"] >= MAX_PENDING_SECONDS:`
- `sources/openviking/openviking_gateway/archives.py:29` `return {**archive, "status": status, "next_check": now + POLL_SECONDS}`

**clear**, body `sources/openviking/openviking_gateway/cache.py:33`; implementation/call-through.

- `sources/openviking/openviking_gateway/cache.py:32` `def clear(self):`
- `sources/openviking/openviking_gateway/cache.py:33` `self.entries.clear()`

**archive_state**, body `sources/openviking/openviking_gateway/client.py:159`; implementation/call-through.

- `sources/openviking/openviking_gateway/client.py:158` `async def archive_state(self, key, uri):`
- `sources/openviking/openviking_gateway/client.py:159` `for marker, state in ((".done", "completed"), (".failed.json", "failed")):`

**delete_user**, body `sources/openviking/openviking_gateway/management.py:288`; implementation/call-through.

- `sources/openviking/openviking_gateway/management.py:286` `@router.delete("/users/{user_id}/data")`
- `sources/openviking/openviking_gateway/management.py:287` `async def delete_user(user_id: str, request: Request):`
- `sources/openviking/openviking_gateway/management.py:288` `who = await caller(request)`

**delete_account**, body `sources/openviking/openviking_gateway/management.py:296`; implementation/call-through.

- `sources/openviking/openviking_gateway/management.py:294` `@router.delete("/account/data")`
- `sources/openviking/openviking_gateway/management.py:295` `async def delete_account(request: Request):`
- `sources/openviking/openviking_gateway/management.py:296` `who = await caller(request)`

**revoke_key**, body `sources/openviking/openviking_gateway/management.py:377`; implementation/call-through.

- `sources/openviking/openviking_gateway/management.py:375` `@router.delete("/keys/{identifier}")`
- `sources/openviking/openviking_gateway/management.py:376` `async def revoke_key(identifier: str, request: Request):`
- `sources/openviking/openviking_gateway/management.py:381` `await management.delete(who.account_id, "keys", identifier)`

**reset**, body `sources/openviking/openviking_gateway/management.py:451`; implementation/call-through.

- `sources/openviking/openviking_gateway/management.py:449` `@router.post("/keys/{identifier}/capture/reset")`
- `sources/openviking/openviking_gateway/management.py:450` `async def reset(identifier: str, data: CaptureReset, request: Request):`
- `sources/openviking/openviking_gateway/management.py:451` `who = await caller(request)`

**delete_object**, body `sources/openviking/openviking_gateway/management.py:341`; implementation/call-through.

- `sources/openviking/openviking_gateway/management.py:339` `@router.delete(f"/{kind}/{{identifier}}")`
- `sources/openviking/openviking_gateway/management.py:340` `async def delete_object(identifier: str, request: Request):`
- `sources/openviking/openviking_gateway/management.py:350` `await management.delete(who.account_id, kind, identifier)`

**delete_user_data**, body `sources/openviking/openviking_gateway/storage.py:559`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:556` `async def delete_user_data(management, store, account, user_id):`
- `sources/openviking/openviking_gateway/storage.py:559` `await management.delete_user(account, user_id)`

**delete_account_data**, body `sources/openviking/openviking_gateway/storage.py:565`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:564` `async def delete_account_data(management, store, account):`
- `sources/openviking/openviking_gateway/storage.py:573` `await management.delete_account(account)`

**delete**, body `sources/openviking/openviking_gateway/storage.py:450`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:449` `async def delete(self, account, kind, identifier):`
- `sources/openviking/openviking_gateway/storage.py:453` `"DELETE FROM objects WHERE account=? AND kind=? AND id=?",`

**delete_user**, body `sources/openviking/openviking_gateway/storage.py:505`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:504` `async def delete_user(self, account, user_id):`
- `sources/openviking/openviking_gateway/storage.py:516` `"DELETE FROM objects WHERE account=? AND kind=? AND id=?", identity`
- `sources/openviking/openviking_gateway/storage.py:519` `"DELETE FROM object_expiry WHERE account=? AND kind=? AND id=?",`
- `sources/openviking/openviking_gateway/storage.py:522` `c.execute("DELETE FROM request_logs WHERE account=? AND user=?", (account, user_id))`

**delete_account**, body `sources/openviking/openviking_gateway/storage.py:544`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:543` `async def delete_account(self, account):`
- `sources/openviking/openviking_gateway/storage.py:547` `c.execute("DELETE FROM objects WHERE account=?", (account,))`
- `sources/openviking/openviking_gateway/storage.py:548` `c.execute("DELETE FROM request_logs WHERE account=?", (account,))`
- `sources/openviking/openviking_gateway/storage.py:549` `c.execute("DELETE FROM object_expiry WHERE account=?", (account,))`

**delete**, body `sources/openviking/openviking_gateway/storage.py:451`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:450` `def delete():`
- `sources/openviking/openviking_gateway/storage.py:453` `"DELETE FROM objects WHERE account=? AND kind=? AND id=?",`

**delete**, body `sources/openviking/openviking_gateway/storage.py:506`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:505` `def delete():`
- `sources/openviking/openviking_gateway/storage.py:516` `"DELETE FROM objects WHERE account=? AND kind=? AND id=?", identity`
- `sources/openviking/openviking_gateway/storage.py:519` `"DELETE FROM object_expiry WHERE account=? AND kind=? AND id=?",`
- `sources/openviking/openviking_gateway/storage.py:522` `c.execute("DELETE FROM request_logs WHERE account=? AND user=?", (account, user_id))`

**delete**, body `sources/openviking/openviking_gateway/storage.py:545`; implementation/call-through.

- `sources/openviking/openviking_gateway/storage.py:544` `def delete():`
- `sources/openviking/openviking_gateway/storage.py:547` `c.execute("DELETE FROM objects WHERE account=?", (account,))`
- `sources/openviking/openviking_gateway/storage.py:548` `c.execute("DELETE FROM request_logs WHERE account=?", (account,))`
- `sources/openviking/openviking_gateway/storage.py:549` `c.execute("DELETE FROM object_expiry WHERE account=?", (account,))`

**reset**, body `sources/openviking/openviking_gateway/tool_loop.py:185`; implementation/call-through.

- `sources/openviking/openviking_gateway/tool_loop.py:183` `def reset(self, anchor, value):`
- `sources/openviking/openviking_gateway/tool_loop.py:185` `start, chain = self.adapter.messages(self.prepared.body), self.prepared.capture_chain`

</details>

## 3. What reaches the agent

`system_prompt_block` supplies usage instructions; `prefetch` joins startup context and query recall as `## OpenViking Context` (`sources/openviking/examples/hermes-plugin/__init__.py:1991`; `sources/openviking/examples/hermes-plugin/__init__.py:2008`). These are separate contributions, so a per-search cap is not a total context cap. Tool output is JSON through the dispatch handler (`sources/openviking/examples/hermes-plugin/__init__.py:3171`). Exact assembly helpers and limit expressions below.

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_tool_schema`, body `sources/openviking/examples/hermes-plugin/__init__.py:485`:

- `sources/openviking/examples/hermes-plugin/__init__.py:485` `return {"name": name, "description": description, "parameters": {"type": "object", "properties": properties, "required": required}}`

`_tool_part`, body `sources/openviking/examples/hermes-plugin/__init__.py:1293`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1293` `return {"type": "tool", "tool_id": tool_id, "tool_name": tool_name, "tool_input": tool_input, **extra, "tool_status": tool_status}`

`_tool_call_id`, body `sources/openviking/examples/hermes-plugin/__init__.py:1297`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1297` `return str(tool_call.get("id") or tool_call.get("tool_call_id") or "")`

`_tool_call_name`, body `sources/openviking/examples/hermes-plugin/__init__.py:1301`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1302` `return str((function.get("name") if isinstance(function, dict) else tool_call.get("name")) or "")`

`_is_openviking_recall_tool_name`, body `sources/openviking/examples/hermes-plugin/__init__.py:1306`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1306` `return str(tool_name or "").strip().lower() in _OPENVIKING_RECALL_TOOL_NAMES`

`_tool_call_input`, body `sources/openviking/examples/hermes-plugin/__init__.py:1310`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1315` `return {}`
- `sources/openviking/examples/hermes-plugin/__init__.py:1317` `return raw_args if isinstance(raw_args, dict) else {"value": raw_args}`
- `sources/openviking/examples/hermes-plugin/__init__.py:1320` `return parsed if isinstance(parsed, dict) else {"value": parsed}`
- `sources/openviking/examples/hermes-plugin/__init__.py:1321` `return {"value": raw_args}`

`_tool_result_status`, body `sources/openviking/examples/hermes-plugin/__init__.py:1325`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1327` `return "error"`
- `sources/openviking/examples/hermes-plugin/__init__.py:1329` `return "completed"`
- `sources/openviking/examples/hermes-plugin/__init__.py:1339` `return "error"`
- `sources/openviking/examples/hermes-plugin/__init__.py:1340` `return "completed"`

`_index_tool_calls`, body `sources/openviking/examples/hermes-plugin/__init__.py:1357`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1373` `return tool_calls_by_id, completed_tool_ids, skipped_tool_ids`

`system_prompt_block`, body `sources/openviking/examples/hermes-plugin/__init__.py:1973`:

- `sources/openviking/examples/hermes-plugin/__init__.py:1974` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:1979` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:1980` `return header + (`
- `sources/openviking/examples/hermes-plugin/__init__.py:1987` `"Prefer one or two focused searches, then read the strongest result URIs. If repeated searches return the same "`
- `sources/openviking/examples/hermes-plugin/__init__.py:1996` `return header + (`
- `sources/openviking/examples/hermes-plugin/__init__.py:1998` `"If repeated searches return the same evidence or no stronger evidence, answer from available evidence and "`

`prefetch`, body `sources/openviking/examples/hermes-plugin/__init__.py:2004`:

- `sources/openviking/examples/hermes-plugin/__init__.py:2006` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2010` `parts.append(self._search_prefetch_context(query_text, session_id=effective_session_id))`
- `sources/openviking/examples/hermes-plugin/__init__.py:2012` `return "## OpenViking Context\n" + "\n\n".join(parts) if parts else ""`

`queue_prefetch`, body `sources/openviking/examples/hermes-plugin/__init__.py:2016`:



`_post_prefetch_search`, body `sources/openviking/examples/hermes-plugin/__init__.py:2030`:

- `sources/openviking/examples/hermes-plugin/__init__.py:2030` `base_payload = {"query": query, "limit": limit, "score_threshold": 0, "context_type": context_type}`
- `sources/openviking/examples/hermes-plugin/__init__.py:2035` `return client.post("/api/v1/search/search", {**base_payload, "session_id": session_id},`
- `sources/openviking/examples/hermes-plugin/__init__.py:2041` `return client.post("/api/v1/search/find", base_payload, timeout=cls._remaining_recall_timeout(deadline, request_timeout))`

`_search_prefetch_context`, body `sources/openviking/examples/hermes-plugin/__init__.py:2044`:

- `sources/openviking/examples/hermes-plugin/__init__.py:2047` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2056` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2058` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2079` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2083` `target_uri.append(f"{user_root}/peers/{sender_peer}/memories")`
- `sources/openviking/examples/hermes-plugin/__init__.py:2087` `target_uri.append(f"{user_root}/peers/{sender_peer}/resources")`
- `sources/openviking/examples/hermes-plugin/__init__.py:2097` `"max_tokens": max(64, min(32000, cfg["max_injected_chars"] // 4)),`
- `sources/openviking/examples/hermes-plugin/__init__.py:2122` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2123` `return str(`
- `sources/openviking/examples/hermes-plugin/__init__.py:2137` `limit=max(cfg["limit"] * 4, 20),`
- `sources/openviking/examples/hermes-plugin/__init__.py:2145` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2147` `selected = self._select_recall_candidates(candidates, query_text, limit=cfg["limit"], score_threshold=cfg["score_threshold"])`
- `sources/openviking/examples/hermes-plugin/__init__.py:2148` `return "\n".join(self._build_prefetch_entries(`
- `sources/openviking/examples/hermes-plugin/__init__.py:2149` `client, selected, prefer_abstract=cfg["prefer_abstract"], max_injected_chars=cfg["max_injected_chars"],`
- `sources/openviking/examples/hermes-plugin/__init__.py:2150` `deadline=deadline, request_timeout=cfg["request_timeout_seconds"], full_read_limit=cfg["full_read_limit"],`
- `sources/openviking/examples/hermes-plugin/__init__.py:2157` `"OpenViking recall timed out (%s; budget_s=%s request_s=%s); no query context injected",`
- `sources/openviking/examples/hermes-plugin/__init__.py:2162` `return ""`

`_session_start_memory_context`, body `sources/openviking/examples/hermes-plugin/__init__.py:2398`:

- `sources/openviking/examples/hermes-plugin/__init__.py:2400` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2404` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2408` `def budgeted_get(path: str, params: dict) -> Any:`
- `sources/openviking/examples/hermes-plugin/__init__.py:2409` `return client.get(path, params=params, timeout=self._remaining_recall_timeout(deadline, request_timeout))`
- `sources/openviking/examples/hermes-plugin/__init__.py:2414` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2417` `profile = self._extract_text_content(budgeted_get("/api/v1/content/read", {"uri": uris[0]}))`
- `sources/openviking/examples/hermes-plugin/__init__.py:2420` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2425` `listings.append(self._extract_memory_listing(budgeted_get("/api/v1/fs/ls", {"uri": uri, **_SESSION_START_LIST_PARAMS})))`
- `sources/openviking/examples/hermes-plugin/__init__.py:2427` `listings.append([])`
- `sources/openviking/examples/hermes-plugin/__init__.py:2430` `return ""`
- `sources/openviking/examples/hermes-plugin/__init__.py:2432` `return self._build_session_start_memory_block(`
- `sources/openviking/examples/hermes-plugin/__init__.py:2433` `profile=profile, preferences=listings[0], entities=listings[1], token_budget=self._profile_token_budget(), uris=uris,`

`_build_prefetch_entries`, body `sources/openviking/examples/hermes-plugin/__init__.py:2486`:

- `sources/openviking/examples/hermes-plugin/__init__.py:2493` `if not (prefer_abstract and has_explicit_summary) and uri and (item.get("level") == 2 or not has_explicit_summary) and full_reads < full_read_limit:`
- `sources/openviking/examples/hermes-plugin/__init__.py:2503` `entry = "\n".join([f"- [{category}]", f"  <uri>{item.get('uri', '')}</uri>", *[f"  {line}" for line in content.splitlines()]])`
- `sources/openviking/examples/hermes-plugin/__init__.py:2505` `if projected_chars <= max_injected_chars:`
- `sources/openviking/examples/hermes-plugin/__init__.py:2506` `entries.append(entry)`
- `sources/openviking/examples/hermes-plugin/__init__.py:2508` `return entries`

`get_tool_schemas`, body `sources/openviking/examples/hermes-plugin/__init__.py:3162`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3162` `return list(_TOOL_SCHEMAS)`

`handle_tool_call`, body `sources/openviking/examples/hermes-plugin/__init__.py:3165`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3166` `return tool_error("OpenViking server not connected")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3169` `return tool_error(f"Unknown tool: {tool_name}")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3171` `return getattr(self, handler)(args)`
- `sources/openviking/examples/hermes-plugin/__init__.py:3173` `return tool_error(str(e))`

`_tool_search`, body `sources/openviking/examples/hermes-plugin/__init__.py:3219`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3221` `return tool_error("query is required")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3222` `payload: Dict[str, Any] = {"query": query, **({"target_uri": args["scope"]} if args.get("scope") else {}), **({"limit": args["limit"]} if args.get("limit") else {})}`
- `sources/openviking/examples/hermes-plugin/__init__.py:3235` `entry["related"] = [r.get("uri") for r in item["relations"][:3]]`
- `sources/openviking/examples/hermes-plugin/__init__.py:3236` `scored_entries.append((raw_score if raw_score is not None else 0.0, entry))`
- `sources/openviking/examples/hermes-plugin/__init__.py:3238` `return json.dumps({"results": formatted, "total": result.get("total", len(formatted))}, ensure_ascii=False)`

`_tool_read`, body `sources/openviking/examples/hermes-plugin/__init__.py:3267`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3274` `return tool_error("uri or uris is required")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3276` `selected = uris[:_READ_BATCH_LIMIT]`
- `sources/openviking/examples/hermes-plugin/__init__.py:3278` `return json.dumps(self._read_uri_payload(selected[0], level), ensure_ascii=False)`
- `sources/openviking/examples/hermes-plugin/__init__.py:3279` `per_item_limit = _READ_BATCH_FULL_LIMIT if len(selected) > 1 and level == "full" else None`
- `sources/openviking/examples/hermes-plugin/__init__.py:3283` `results.append(self._read_uri_payload(uri, level, limit=per_item_limit))`
- `sources/openviking/examples/hermes-plugin/__init__.py:3285` `results.append({"uri": uri, "level": level, "error": str(e)})`
- `sources/openviking/examples/hermes-plugin/__init__.py:3286` `return json.dumps({"level": level, "results": results, "requested": len(uris), "returned": len(results),`

`_tool_browse`, body `sources/openviking/examples/hermes-plugin/__init__.py:3290`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3299` `for e in raw_entries[:50]]`
- `sources/openviking/examples/hermes-plugin/__init__.py:3300` `return json.dumps({"path": path, "entries": entries}, ensure_ascii=False)`
- `sources/openviking/examples/hermes-plugin/__init__.py:3301` `return json.dumps(result, ensure_ascii=False)`

`_tool_remember`, body `sources/openviking/examples/hermes-plugin/__init__.py:3305`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3307` `return tool_error("content is required")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3310` `return tool_error("OpenViking server not connected")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3312` `session_id = f"hermes-remember-{uuid.uuid4().hex[:12]}"`
- `sources/openviking/examples/hermes-plugin/__init__.py:3316` `return tool_error(`
- `sources/openviking/examples/hermes-plugin/__init__.py:3329` `return failure(f"Memory message submission failed for session {session_id}: {e}", stage="message", message_status="unknown")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3334` `return failure(f"Memory message was accepted, but commit failed for session {session_id}: {e}", stage="commit", message_status="accepted")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3336` `return json.dumps({`

`_tool_forget`, body `sources/openviking/examples/hermes-plugin/__init__.py:3345`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3348` `return tool_error(error)`
- `sources/openviking/examples/hermes-plugin/__init__.py:3353` `return json.dumps(payload, ensure_ascii=False)`

`_tool_add_resource`, body `sources/openviking/examples/hermes-plugin/__init__.py:3356`:

- `sources/openviking/examples/hermes-plugin/__init__.py:3360` `return tool_error("url is required")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3362` `return tool_error("Cannot specify both 'to' and 'parent'")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3373` `return tool_error(f"Unsupported non-local file URI: {url}")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3382` `return tool_error(f"Local resource path does not exist: {url}")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3391` `return tool_error(str(exc))`
- `sources/openviking/examples/hermes-plugin/__init__.py:3395` `return tool_error(f"Unsupported local resource path: {url}")`
- `sources/openviking/examples/hermes-plugin/__init__.py:3401` `return json.dumps({`

`flush_tool_parts`, body `sources/openviking/examples/hermes-plugin/__init__.py:2553`:



</details>

## 4. Data access boundaries

Direct access `yes`: the same API is callable without the adapter; local backing paths are also accessible to their OS owner. API keys/model keys are configuration-specific, not protected by URI validation in the adapter (`sources/openviking/examples/hermes-plugin/__init__.py:3346`). Profile `.env`, client/server config and local runtime paths/modes are in the source ledger. Accountless local mode `yes` with locally configured models; no claim that model downloads or defaults are offline. Telemetry: no universal default established across optional server/model dependencies.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/openviking/crates/ov_cli/src/config.rs:9` `pub const DEFAULT_CUSTOM_URL: &str = "http://127.0.0.1:1933";`
- `sources/openviking/crates/ov_cli/src/config.rs:449` `url: "http://not-the-default".to_string(),`
- `sources/openviking/crates/ov_cli/src/config.rs:521` `assert_eq!(config.url, "http://127.0.0.1:1933");`
- `sources/openviking/crates/ov_cli/src/config.rs:528` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:551` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:572` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:632` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:761` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:830` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:856` `"url": "http://127.0.0.1:1933"`
- `sources/openviking/crates/ov_cli/src/config.rs:868` `"url": "http://127.0.0.1:1933",`
- `sources/openviking/crates/ov_cli/src/config.rs:881` `"url": "http://127.0.0.1:1933",`

- `sources/openviking/crates/ov_cli/src/config_agent.rs:1069` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1113` `let config = sample_config("http://127.0.0.1:1933", Some("key"));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1141` `&sample_config("http://127.0.0.1:1933", Some("old")),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1149` `&sample_config("http://127.0.0.1:1933", Some("new")),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1162` `"http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1192` `"https://ov.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1208` `"http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1222` `let mut config = sample_config("http://127.0.0.1:1933", None);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1235` `let mut config = sample_config("http://127.0.0.1:1933", None);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1248` `"http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1257` `let mut edit_config = sample_config("http://127.0.0.1:1933", None);`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1304` `let mut config = sample_config("http://127.0.0.1:1933", Some("user-key"));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1325` `let old_config = sample_config("http://127.0.0.1:1933", Some("old"));`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1334` `&sample_config("http://127.0.0.1:1933", Some("new")),`
- `sources/openviking/crates/ov_cli/src/config_agent.rs:1374` `&sample_config("http://127.0.0.1:1933", Some("key")),`

- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:491` `assert!(plain.contains("Server        http://127.0.0.1:1933"));`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:540` `assert!(!plain.contains("http://"));`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:541` `assert!(!plain.contains("https://"));`
- `sources/openviking/crates/ov_cli/src/config_command_ui.rs:546` `url: "http://127.0.0.1:1933".to_string(),`

- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:16` `pub const OPENVIKING_SERVICE_URL: &str = "https://api.vikingdb.cn-beijing.volces.com/openviking";`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:546` `return format!("http://[::1]:{DEFAULT_CUSTOM_PORT}");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:551` `return format!("http://[::1]:{port}");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:556` `return format!("http://[::1]:{port}");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:569` `format!("http://{host}:{DEFAULT_CUSTOM_PORT}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:571` `format!("http://{host}:{port}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:821` `permissions.set_mode(0o600);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:872` `let config = sample_config("http://selected", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:875` `fs::write(dir.path().join("ovcli.conf"), r#"{"url":"http://default"}"#).unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:881` `assert_eq!(store.load_active().unwrap().unwrap().url, "http://selected");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:887` `.save_active_config(&sample_config("http://edited", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:891` `"http://edited"`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:897` `"http://default"`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:905` `fs::write(&default, r#"{"url":"http://default"}"#).unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:908` `.save_named_config("next", &sample_config("http://next", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:911` `assert_eq!(store.load_active().unwrap().unwrap().url, "http://next");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:914` `"http://default"`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:952` `"url": "http://old", "api_key": "old-key", "plugin": plugin_section(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:958` `super::write_config_file(&path, &sample_config("http://new", Some("new-key"))).unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:961` `assert_eq!(written["url"], "http://new");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:975` `"url": "http://local", "api_key": "old-key", "gateway_token": "old-token",`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:981` `super::write_config_file(&path, &sample_config("http://local", Some("new-key"))).unwrap();`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1000` `serde_json::json!({ "url": "http://local", "plugin": plugin_section() }).to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1005` `.save_edited_config("before", "after", &sample_config("http://local", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1023` `serde_json::json!({ "url": "http://old", "plugin": plugin_section() }).to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1028` `serde_json::json!({ "url": "http://other" }).to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1037` `assert_eq!(active["url"], "http://other");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1044` `let mut config = sample_config("http://local", Some("api-secret"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1083` `&sample_config("http://active", None),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1087` `&sample_config("http://backup", None),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1091` `&sample_config("http://local", None),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1100` `assert_eq!(configs[0].config.url, "http://local");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1110` `&sample_config("http://local", None),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1128` `let config = sample_config("http://local", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1228` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1235` `assert_eq!(without_key.url, "http://127.0.0.1:1933");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1258` `url: "https://openviking.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1273` `("127.0.0.1", "http://127.0.0.1:1933"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1274` `("localhost", "http://localhost:1933"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1275` `("::1", "http://[::1]:1933"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1276` `("[::1]", "http://[::1]:1933"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1277` `("::1:1944", "http://[::1]:1944"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1278` `("[::1]:1944", "http://[::1]:1944"),`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1393` `let original = sample_config("http://old", Some("old-key"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1401` `let edited = sample_config("http://new", Some("new-key"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1411` `assert_eq!(saved.url, "http://new");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1412` `assert_eq!(active.url, "http://new");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1420` `let original = sample_config("http://local", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1431` `let edited = sample_config("http://copy", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1441` `assert_eq!(active.url, "http://local");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1442` `assert_eq!(copy.url, "http://copy");`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1450` `.save_named_config("local", &sample_config("http://local", None))`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1464` `let config = sample_config("http://local", None);`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1517` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1548` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1600` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/store.rs:1642` `format!("http://{addr}")`

- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:35` `"https://console.volcengine.com/vikingdb/openviking/region:openviking+cn-beijing";`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6256` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6314` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6441` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6491` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6554` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:6863` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7063` `url: "https://existing.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7071` `url: "https://replacement.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7082` `assert_eq!(saved.url, "https://existing.example.com");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7228` `url: "http://127.0.0.1:1933".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7286` `url: "https://inactive.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7295` `url: "https://active.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7305` `assert_eq!(context.config.url, "https://active.example.com");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7311` `url: "https://legacy.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7321` `assert_eq!(legacy_context.config.url, "https://legacy.example.com");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7327` `url: "https://unnamed.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7339` `assert_eq!(unnamed_context.config.url, "https://unnamed.example.com");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7351` `assert_eq!(config.url, "http://127.0.0.1:1933");`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7364` `strip_ansi(&root_api_key_helper_lines("https://openviking.example.com").join("\n"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7367` `assert!(text.contains("https://openviking.example.com"));`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7385` `url: "https://openviking.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7418` `"http://127.0.0.1:1933"`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7422` `url: "https://openviking.example.com".to_string(),`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7431` `"https://openviking.example.com"`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7829` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7881` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:7936` `format!("http://{addr}")`
- `sources/openviking/crates/ov_cli/src/config_wizard/wizard.rs:8105` `config.url = "http://127.0.0.1:1933".to_string();`

- `sources/openviking/examples/hermes-plugin/__init__.py:62` `_DEFAULT_ENDPOINT = "http://127.0.0.1:1933"`
- `sources/openviking/examples/hermes-plugin/__init__.py:63` `_OPENVIKING_SERVICE_ENDPOINT = "https://api.vikingdb.cn-beijing.volces.com/openviking"`
- `sources/openviking/examples/hermes-plugin/__init__.py:121` `_REMOTE_RESOURCE_PREFIXES = ("http://", "https://", "git@", "ssh://", "git://")`
- `sources/openviking/examples/hermes-plugin/__init__.py:343` `self._account = (get_secret("OPENVIKING_ACCOUNT", "") if account is _IDENTITY_UNSET or account is None else account) or "default"`
- `sources/openviking/examples/hermes-plugin/__init__.py:344` `self._user = (get_secret("OPENVIKING_USER", "") if user is _IDENTITY_UNSET or user is None else user) or "default"`
- `sources/openviking/examples/hermes-plugin/__init__.py:345` `self._agent = (get_secret("OPENVIKING_AGENT", "") or _DEFAULT_AGENT) if agent is _IDENTITY_UNSET or agent is None else agent`
- `sources/openviking/examples/hermes-plugin/__init__.py:350` `_parse_extra_headers(get_secret(_EXTRA_HEADERS_ENV))`
- `sources/openviking/examples/hermes-plugin/__init__.py:681` `return Path.home() / _OVCLI_DEFAULT_RELATIVE_PATH`
- `sources/openviking/examples/hermes-plugin/__init__.py:685` `chosen = (os.environ if env is None else env).get(_OVCLI_CONFIG_ENV, "").strip() or config_path`
- `sources/openviking/examples/hermes-plugin/__init__.py:754` `candidate = f"http://{trimmed}:1933"`
- `sources/openviking/examples/hermes-plugin/__init__.py:756` `candidate = "http://[::1]:1933"`
- `sources/openviking/examples/hermes-plugin/__init__.py:758` `candidate = f"http://[::1]:{trimmed.rsplit(':', 1)[1]}"`
- `sources/openviking/examples/hermes-plugin/__init__.py:760` `candidate = trimmed if "://" in trimmed else f"http://{trimmed}"`
- `sources/openviking/examples/hermes-plugin/__init__.py:764` `raise ValueError("OpenViking endpoints must use http:// or https:// with a host.")`
- `sources/openviking/examples/hermes-plugin/__init__.py:838` `env_path = os.environ.get(_OVCLI_CONFIG_ENV, "").strip()`
- `sources/openviking/examples/hermes-plugin/__init__.py:926` `for key, value in os.environ.items():`
- `sources/openviking/examples/hermes-plugin/__init__.py:955` `under multiplexing ''os.environ'' is the DEFAULT profile's .env, and a raw read`
- `sources/openviking/examples/hermes-plugin/__init__.py:970` `value = get_secret(f"OPENVIKING_{key.upper()}") if env is None else env.get(f"OPENVIKING_{key.upper()}")`
- `sources/openviking/examples/hermes-plugin/__init__.py:977` `api_key_env = get_secret("OPENVIKING_API_KEY") if env is None else env.get("OPENVIKING_API_KEY")`
- `sources/openviking/examples/hermes-plugin/__init__.py:982` `extra_headers_raw = get_secret(_EXTRA_HEADERS_ENV) if env is None else env.get(_EXTRA_HEADERS_ENV)`
- `sources/openviking/examples/hermes-plugin/__init__.py:994` `"""chmod 0600 a secret-bearing file; with ''create'' also pre-create it BEFORE writing`
- `sources/openviking/examples/hermes-plugin/__init__.py:995` `(write-then-chmod leaves a window where the fresh file is world-readable under the umask)."""`
- `sources/openviking/examples/hermes-plugin/__init__.py:998` `os.close(os.open(str(path), os.O_CREAT | os.O_WRONLY, 0o600))`
- `sources/openviking/examples/hermes-plugin/__init__.py:999` `path.chmod(stat.S_IRUSR | stat.S_IWUSR)`
- `sources/openviking/examples/hermes-plugin/__init__.py:1232` `subprocess.Popen([server_cmd, "--host", host, "--port", str(port)], stdout=log_file, stderr=log_file,`
- `sources/openviking/examples/hermes-plugin/__init__.py:1233` `stdin=subprocess.DEVNULL, start_new_session=True, env=child_env)`
- `sources/openviking/examples/hermes-plugin/__init__.py:1542` `if get_secret("OPENVIKING_ENDPOINT", ""):`
- `sources/openviking/examples/hermes-plugin/__init__.py:1596` `elif env_overrides := [key for key in _OPENVIKING_ENV_KEYS if key in os.environ]:`
- `sources/openviking/examples/hermes-plugin/__init__.py:1834` `''/reload'' only refreshes ''os.environ''; the provider instance is not`
- `sources/openviking/examples/hermes-plugin/__init__.py:1838` `''/reload'' only refreshes ''os.environ'' , the existing provider instance is not re-initialized ,`
- `sources/openviking/examples/hermes-plugin/__init__.py:2197` `env_value = get_secret(spec["env_var"]) if env is None else env.get(spec["env_var"])`
- `sources/openviking/examples/hermes-plugin/__init__.py:2226` `override = get_secret(_SETTING_SPECS[key]["env_var"]) if env is None else env.get(_SETTING_SPECS[key]["env_var"])`
- `sources/openviking/examples/hermes-plugin/__init__.py:2864` `"connection_key": scope.connection_key}, mode=0o600)`

- `sources/openviking/examples/hermes-plugin/_setup.py:296` `if os.environ.get(ov._OVCLI_CONFIG_ENV, "").strip() or ovcli_path.expanduser() != ov._default_ovcli_config_path().expanduser():`
- `sources/openviking/examples/hermes-plugin/_setup.py:304` `os.environ.pop(key, None)`
- `sources/openviking/examples/hermes-plugin/_setup.py:322` `os.environ.update(writes)`
- `sources/openviking/examples/hermes-plugin/_setup.py:324` `os.environ.pop(key, None)`
- `sources/openviking/examples/hermes-plugin/_setup.py:402` `ov.atomic_json_write(path, ov._ovcli_data_from_connection_values(values), mode=0o600)`
- `sources/openviking/examples/hermes-plugin/_setup.py:540` `os.environ.pop("OPENVIKING_RECALL_SCOPE", None)`

- `sources/openviking/examples/hermes-plugin/local_packages.py:20` `_CPP = "https://github.com/abetlen/llama-cpp-python/releases/download/"`
- `sources/openviking/examples/hermes-plugin/local_packages.py:82` `response = httpx.get("https://pypi.org/pypi/openviking/json", timeout=30)`

- `sources/openviking/examples/hermes-plugin/local_server.py:107` `path.chmod(0o600)`
- `sources/openviking/examples/hermes-plugin/local_server.py:225` `atomic_json_write(self.record_path, record, mode=0o600)`
- `sources/openviking/examples/hermes-plugin/local_server.py:254` `atomic_json_write(self.paths.server_config, config, mode=0o600)`
- `sources/openviking/examples/hermes-plugin/local_server.py:256` `self.paths.ovcli_config, f"http://127.0.0.1:{config['server']['port']}", config`
- `sources/openviking/examples/hermes-plugin/local_server.py:302` `endpoint = f"http://127.0.0.1:{port}"`
- `sources/openviking/examples/hermes-plugin/local_server.py:304` `atomic_json_write(self.paths.server_config, config, mode=0o600)`
- `sources/openviking/examples/hermes-plugin/local_server.py:324` `mode=0o600,`

- `sources/openviking/examples/hermes-plugin/native_memory_mirror.py:207` `atomic_json_write(path, registry, mode=0o600)`

- `sources/openviking/examples/hermes-plugin/quick_local.py:178` `mode=0o600,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:423` `endpoint=f"http://127.0.0.1:{validation_port}",`
- `sources/openviking/examples/hermes-plugin/quick_local.py:454` `endpoint = f"http://127.0.0.1:{port}"`
- `sources/openviking/examples/hermes-plugin/quick_local.py:540` `paths.runtime_validation_pending_marker, {"validation_pending": True}, mode=0o600`
- `sources/openviking/examples/hermes-plugin/quick_local.py:542` `atomic_json_write(receipt, requirements, mode=0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:576` `result = subprocess.run(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:580` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:602` `atomic_json_write(config_path, validation_config, mode=0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:717` `result = subprocess.run(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:721` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:722` `stdout=subprocess.PIPE,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:723` `stderr=subprocess.PIPE,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:736` `except subprocess.TimeoutExpired as exc:`
- `sources/openviking/examples/hermes-plugin/quick_local.py:775` `log_path.touch(mode=0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:777` `log_path.chmod(0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:787` `result = subprocess.run(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:797` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:803` `except subprocess.TimeoutExpired as exc:`
- `sources/openviking/examples/hermes-plugin/quick_local.py:808` `except (InvalidVersion, OSError, subprocess.SubprocessError):`
- `sources/openviking/examples/hermes-plugin/quick_local.py:827` `log_path.touch(mode=0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:829` `log_path.chmod(0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:832` `result = subprocess.run(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:835` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:837` `stderr=subprocess.STDOUT,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:846` `except subprocess.TimeoutExpired as exc:`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1000` `mode=0o600,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1038` `) -> subprocess.Popen:`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1051` `log_path.touch(mode=0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1052` `log_path.chmod(0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1054` `from hermes_cli._subprocess_compat import windows_detach_popen_kwargs`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1056` `popen_kwargs: dict[str, Any] = windows_detach_popen_kwargs()`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1075` `return subprocess.Popen(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1079` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1088` `return subprocess.Popen(`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1092` `stdin=subprocess.DEVNULL,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1102` `process: Optional[subprocess.Popen] = None,`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1116` `def _stop_process(process: subprocess.Popen) -> bool:`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1193` `path.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1195` `path.chmod(0o700)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1213` `path.chmod(0o600)`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1242` `endpoint = f"http://{endpoint}"`
- `sources/openviking/examples/hermes-plugin/quick_local.py:1249` `return f"http://{host}:{parsed.port or _DEFAULT_PORT}"`

- `sources/openviking/openviking/config/config_file.py:257` `os.chmod(temporary, mode)`
- `sources/openviking/openviking/config/config_file.py:259` `os.fchmod(stream.fileno(), mode)`
- `sources/openviking/openviking/config/config_file.py:288` `lock_fd = os.open(path.with_name(f".{path.name}.studio.lock"), os.O_CREAT | os.O_RDWR, 0o600)`
- `sources/openviking/openviking/config/config_file.py:316` `_atomic_write(path.with_name(f"{path.name}.studio.bak"), data, 0o600)`

- `sources/openviking/openviking/config/embedding.py:239` `from openviking.telemetry import bind_telemetry_stage`
- `sources/openviking/openviking/config/embedding.py:247` `with bind_telemetry_stage("embed_query" if is_query else "embed_resource"):`

- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:5` `Event collector: TelemetryBridgeCollector.`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:7` `This collector converts the aggregated telemetry summary of a single operation/request`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:11` `- A single 'telemetry.summary' event per operation (best-effort).`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:18` `- The telemetry summary is already aggregated; we avoid adding labels that would cause`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:33` `class TelemetryBridgeCollector(EventMetricCollector):`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:35` `Translate aggregated telemetry summaries into low-cardinality Prometheus metrics.`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:120` `SUPPORTED_EVENTS: ClassVar[frozenset[str]] = frozenset({"telemetry.summary"})`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:130` `"""Expand telemetry token payloads into '(stage, token_type, value)' tuples."""`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:176` `Telemetry summaries are pushed in from runtime code, so scrape-time collection has no`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:183` `Translate the supported telemetry summary event into derived metric writes.`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:191` `self.record_telemetry_summary(registry, summary)`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:193` `def record_telemetry_summary(self, registry, summary: Mapping[str, Any]) -> None:`
- `sources/openviking/openviking/metrics/collectors/telemetry_bridge.py:195` `Record counters, histograms, and gauges derived from one aggregated telemetry summary.`

- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:5` `Telemetry bridge event datasource.`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:7` `This datasource emits the aggregated request/operation telemetry summary as a single`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:8` `best-effort event. 'TelemetryBridgeCollector' consumes that event and translates it into`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:19` `class TelemetryBridgeEventDataSource(EventMetricDataSource):`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:21` `Emit aggregated telemetry summaries for later translation by 'TelemetryBridgeCollector'.`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:23` `This datasource forms the bridge between the runtime telemetry subsystem and Prometheus`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:30` `Emit a best-effort telemetry summary event using a plain-dictionary payload copy.`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:33` `mutability or concrete type of the source telemetry object.`
- `sources/openviking/openviking/metrics/datasources/telemetry_bridge.py:35` `EventMetricDataSource._emit("telemetry.summary", {"summary": dict(summary)})`

- `sources/openviking/openviking/scheduling/expiry/config.py:69` `return os.environ[self.password_env]`

- `sources/openviking/openviking/server/config.py:195` `"""OpenTelemetry exporter configuration."""`
- `sources/openviking/openviking/server/config.py:360` `bot_api_url: str = "http://localhost:18790"  # Vikingbot OpenAPIChannel URL (default port)`
- `sources/openviking/openviking/server/config.py:442` `return f"http://{host}:{port}"`
- `sources/openviking/openviking/server/config.py:503` `f"See: https://openviking.ai/docs"`

- `sources/openviking/openviking/server/oauth/storage.py:142` `def __init__(self, db_path: Path) -> None:`
- `sources/openviking/openviking/server/oauth/storage.py:143` `self._db_path = Path(db_path)`
- `sources/openviking/openviking/server/oauth/storage.py:151` `self._db_path.parent.mkdir(parents=True, exist_ok=True)`
- `sources/openviking/openviking/server/oauth/storage.py:153` `self._db_path,`

- `sources/openviking/openviking/server/telemetry.py:3` `"""HTTP router helpers for operation telemetry."""`
- `sources/openviking/openviking/server/telemetry.py:9` `from openviking.telemetry import TelemetryRequest, TelemetrySelection`
- `sources/openviking/openviking/server/telemetry.py:10` `from openviking.telemetry.execution import (`
- `sources/openviking/openviking/server/telemetry.py:11` `TelemetryExecutionResult,`
- `sources/openviking/openviking/server/telemetry.py:12` `parse_telemetry_selection,`
- `sources/openviking/openviking/server/telemetry.py:13` `run_with_telemetry,`
- `sources/openviking/openviking/server/telemetry.py:17` `def resolve_selection(telemetry: TelemetryRequest) -> TelemetrySelection:`
- `sources/openviking/openviking/server/telemetry.py:18` `"""Validate a router telemetry request without starting execution."""`
- `sources/openviking/openviking/server/telemetry.py:19` `return parse_telemetry_selection(telemetry)`
- `sources/openviking/openviking/server/telemetry.py:25` `telemetry: TelemetryRequest,`
- `sources/openviking/openviking/server/telemetry.py:27` `) -> TelemetryExecutionResult[Any]:`
- `sources/openviking/openviking/server/telemetry.py:28` `"""Execute a router operation with request-scoped telemetry."""`
- `sources/openviking/openviking/server/telemetry.py:29` `return await run_with_telemetry(`
- `sources/openviking/openviking/server/telemetry.py:31` `telemetry=telemetry,`

- `sources/openviking/openviking/storage/collection_schemas.py:45` `from openviking.telemetry import bind_ingestion_source_type, bind_telemetry, resolve_telemetry`
- `sources/openviking/openviking/storage/collection_schemas.py:487` `_request_stats_by_telemetry_id: Dict[str, RequestQueueStats] = {}`
- `sources/openviking/openviking/storage/collection_schemas.py:516` `telemetry_id: str,`
- `sources/openviking/openviking/storage/collection_schemas.py:521` `if not telemetry_id:`
- `sources/openviking/openviking/storage/collection_schemas.py:524` `stats = cls._request_stats_by_telemetry_id.setdefault(telemetry_id, RequestQueueStats())`
- `sources/openviking/openviking/storage/collection_schemas.py:528` `cls._request_stats_order.append(telemetry_id)`
- `sources/openviking/openviking/storage/collection_schemas.py:530` `old_telemetry_id = cls._request_stats_order.pop(0)`
- `sources/openviking/openviking/storage/collection_schemas.py:532` `old_telemetry_id != telemetry_id`
- `sources/openviking/openviking/storage/collection_schemas.py:533` `and old_telemetry_id in cls._request_stats_by_telemetry_id`
- `sources/openviking/openviking/storage/collection_schemas.py:535` `cls._request_stats_by_telemetry_id.pop(old_telemetry_id, None)`
- `sources/openviking/openviking/storage/collection_schemas.py:538` `def consume_request_stats(cls, telemetry_id: str) -> Optional[RequestQueueStats]:`
- `sources/openviking/openviking/storage/collection_schemas.py:539` `if not telemetry_id:`
- `sources/openviking/openviking/storage/collection_schemas.py:542` `return cls._request_stats_by_telemetry_id.pop(telemetry_id, None)`
- `sources/openviking/openviking/storage/collection_schemas.py:559` `f"{log_correlation(telemetry_id=embedding_msg.telemetry_id, message_id=embedding_msg.id)} "`
- `sources/openviking/openviking/storage/collection_schemas.py:696` `collector = resolve_telemetry(embedding_msg.telemetry_id)`
- `sources/openviking/openviking/storage/collection_schemas.py:697` `telemetry_ctx = bind_telemetry(collector) if collector is not None else nullcontext()`
- `sources/openviking/openviking/storage/collection_schemas.py:699` `with telemetry_ctx:`
- `sources/openviking/openviking/storage/collection_schemas.py:702` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:706` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:712` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:719` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:759` `self._merge_request_stats(embedding_msg.telemetry_id, requeue_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:785` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:797` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:813` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:817` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:845` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:940` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:950` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:957` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:964` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:969` `self._merge_request_stats(embedding_msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/collection_schemas.py:984` `self._merge_request_stats(embedding_msg.telemetry_id, error_count=1)`

- `sources/openviking/openviking/storage/content_write.py:82` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/content_write.py:343` `telemetry_id = get_current_telemetry().telemetry_id`
- `sources/openviking/openviking/storage/content_write.py:354` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:368` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:591` `telemetry_id = get_current_telemetry().telemetry_id`
- `sources/openviking/openviking/storage/content_write.py:600` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:650` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:709` `await self._wait_for_request(telemetry_id=telemetry_id)`
- `sources/openviking/openviking/storage/content_write.py:873` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:948` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:996` `telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/content_write.py:1007` `telemetry_id=telemetry.telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:1039` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:1057` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:1565` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:1583` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:1640` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/content_write.py:1670` `await self._wait_for_request(telemetry_id=telemetry_id)`
- `sources/openviking/openviking/storage/content_write.py:1762` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:1791` `await self._wait_for_request(telemetry_id=telemetry_id)`
- `sources/openviking/openviking/storage/content_write.py:2095` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:2097` `del telemetry_id`
- `sources/openviking/openviking/storage/content_write.py:2124` `telemetry_id: str,`
- `sources/openviking/openviking/storage/content_write.py:2173` `telemetry_id=telemetry_id,`

- `sources/openviking/openviking/storage/context_update_execution.py:70` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/context_update_execution.py:75` `telemetry_id = get_current_telemetry().telemetry_id`
- `sources/openviking/openviking/storage/context_update_execution.py:88` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/context_update_execution.py:128` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/context_update_execution.py:151` `telemetry_id=telemetry_id,`

- `sources/openviking/openviking/storage/queuefs/add_resource_msg.py:28` `telemetry_id: Optional[str] = None`
- `sources/openviking/openviking/storage/queuefs/add_resource_msg.py:177` `telemetry_id=str(data.get("telemetry_id"))`
- `sources/openviking/openviking/storage/queuefs/add_resource_msg.py:178` `if isinstance(data.get("telemetry_id"), str)`

- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:21` `from openviking.telemetry import (`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:22` `OperationTelemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:23` `bind_telemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:24` `register_telemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:25` `resolve_telemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:26` `unregister_telemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:28` `from openviking.telemetry.resource_summary import record_resource_queue_metrics`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:147` `telemetry_id = msg.telemetry_id or ""`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:183` `unregister_telemetry(telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:205` `unregister_telemetry(telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:212` `telemetry = resolve_telemetry(telemetry_id) if telemetry_id else None`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:213` `if telemetry is None:`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:214` `telemetry = OperationTelemetry(operation="add_resource_job", enabled=True)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:215` `if telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:216` `telemetry.telemetry_id = telemetry_id`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:218` `telemetry_id = telemetry.telemetry_id`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:219` `register_telemetry(telemetry)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:232` `with bind_execution_context(), bind_telemetry(telemetry):`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:240` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:271` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:314` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:315` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:319` `telemetry.set("resource.total.duration_ms", telemetry.elapsed_ms())`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:322` `_snapshot = telemetry.finish()`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:324` `result["telemetry"] = _snapshot.to_dict(include_summary=True)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:335` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:376` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:389` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:414` `unregister_telemetry(telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/add_resource_processor.py:441` `unregister_telemetry(msg.telemetry_id or "")`

- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:102` `telemetry_id: str = ""`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:113` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:128` `self.telemetry_id = telemetry_id`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:241` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:248` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:262` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:276` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:285` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:290` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:294` `def noop(cls, *, context_data: Dict[str, Any], telemetry_id: str = "") -> "EmbeddingMsg":`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:298` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:338` `"telemetry_id": self.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:409` `telemetry_id=str(data.get("telemetry_id") or ""),`
- `sources/openviking/openviking/storage/queuefs/embedding_msg.py:420` `telemetry_id=str(data.get("telemetry_id") or ""),`

- `sources/openviking/openviking/storage/queuefs/embedding_msg_converter.py:14` `from openviking.telemetry import get_current_telemetry, get_ingestion_source_type`
- `sources/openviking/openviking/storage/queuefs/embedding_msg_converter.py:28` `telemetry_id: str | None = None,`
- `sources/openviking/openviking/storage/queuefs/embedding_msg_converter.py:99` `telemetry_id=(`
- `sources/openviking/openviking/storage/queuefs/embedding_msg_converter.py:100` `get_current_telemetry().telemetry_id if telemetry_id is None else telemetry_id`

- `sources/openviking/openviking/storage/queuefs/reindex_msg.py:26` `telemetry_id: Optional[str] = None`
- `sources/openviking/openviking/storage/queuefs/reindex_msg.py:61` `telemetry_id=data.get("telemetry_id")`
- `sources/openviking/openviking/storage/queuefs/reindex_msg.py:62` `if isinstance(data.get("telemetry_id"), str)`

- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:16` `from openviking.telemetry import (`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:17` `OperationTelemetry,`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:18` `bind_telemetry,`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:19` `register_telemetry,`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:20` `resolve_telemetry,`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:21` `unregister_telemetry,`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:99` `unregister_telemetry(msg.telemetry_id or "")`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:113` `unregister_telemetry(msg.telemetry_id or "")`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:116` `telemetry_id = msg.telemetry_id or ""`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:117` `telemetry = resolve_telemetry(telemetry_id) if telemetry_id else None`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:118` `if telemetry is None:`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:119` `telemetry = OperationTelemetry(operation="reindex_job", enabled=True)`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:120` `if telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:121` `telemetry.telemetry_id = telemetry_id`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:123` `telemetry_id = telemetry.telemetry_id`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:124` `register_telemetry(telemetry)`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:137` `with bind_telemetry(telemetry):`
- `sources/openviking/openviking/storage/queuefs/reindex_processor.py:178` `unregister_telemetry(telemetry_id)`

- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:34` `from openviking.telemetry import bind_ingestion_source_type, bind_telemetry, get_current_telemetry`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:200` `telemetry_id: str | None = None,`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:248` `self._telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:249` `self._telemetry_id = telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:532` `bind_telemetry(self._telemetry),`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1197` `if self._telemetry_id is not None:`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1198` `vectorize_kwargs["telemetry_id"] = self._telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1607` `if self._telemetry_id is not None:`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1608` `directory_vector_kwargs["telemetry_id"] = self._telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1663` `{"telemetry_id": self._telemetry_id}`
- `sources/openviking/openviking/storage/queuefs/semantic_executor.py:1664` `if self._telemetry_id is not None`

- `sources/openviking/openviking/storage/queuefs/semantic_msg.py:62` `telemetry_id: str = ""`
- `sources/openviking/openviking/storage/queuefs/semantic_msg.py:104` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/semantic_msg.py:144` `self.telemetry_id = telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_msg.py:228` `telemetry_id=data.get("telemetry_id", ""),`

- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:63` `from openviking.telemetry import (`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:64` `bind_telemetry,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:65` `bind_telemetry_stage,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:67` `from openviking.telemetry.span_models import create_root_span_attributes`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:127` `_tree_stats_by_telemetry_id: Dict[str, SemanticTreeStats] = {}`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:130` `_request_stats_by_telemetry_id: Dict[str, RequestQueueStats] = {}`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:167` `def _cache_tree_stats(cls, telemetry_id: str, uri: str, stats: SemanticTreeStats) -> None:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:169` `if telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:170` `previous = cls._tree_stats_by_telemetry_id.get(telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:180` `cls._tree_stats_by_telemetry_id[telemetry_id] = stats`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:182` `cls._tree_stats_order.append((telemetry_id, uri))`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:184` `old_telemetry_id, old_uri = cls._tree_stats_order.pop(0)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:185` `if old_telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:186` `cls._tree_stats_by_telemetry_id.pop(old_telemetry_id, None)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:192` `telemetry_id: str = "",`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:196` `if telemetry_id and telemetry_id in cls._tree_stats_by_telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:197` `stats = cls._tree_stats_by_telemetry_id.pop(telemetry_id, None)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:208` `telemetry_id: str,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:213` `if not telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:216` `stats = cls._request_stats_by_telemetry_id.setdefault(telemetry_id, RequestQueueStats())`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:220` `cls._request_stats_order.append(telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:222` `old_telemetry_id = cls._request_stats_order.pop(0)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:223` `if old_telemetry_id != telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:224` `cls._request_stats_by_telemetry_id.pop(old_telemetry_id, None)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:227` `def consume_request_stats(cls, telemetry_id: str) -> Optional[RequestQueueStats]:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:228` `if not telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:231` `return cls._request_stats_by_telemetry_id.pop(telemetry_id, None)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:340` `self._merge_request_stats(msg.telemetry_id, requeue_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:343` `self._merge_request_stats(msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:551` `self._merge_request_stats(msg.telemetry_id, requeue_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:553` `collector = work.resolve_telemetry()`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:554` `telemetry_ctx = bind_telemetry(collector) if collector is not None else nullcontext()`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:555` `with telemetry_ctx:`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:559` `request_id=msg.telemetry_id or msg.id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:643` `telemetry_id=msg.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:648` `msg.telemetry_id, run_uri, executor.get_stats()`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:767` `telemetry_id=msg.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:772` `msg.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:797` `self._merge_request_stats(msg.telemetry_id, processed=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:837` `self._merge_request_stats(msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:855` `self._merge_request_stats(msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:868` `self._merge_request_stats(msg.telemetry_id, error_count=1)`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1070` `telemetry_id=msg.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1139` `telemetry_id=msg.telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1372` `with bind_telemetry_stage("semantic_execute"):`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1730` `with bind_telemetry_stage("semantic_execute"):`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1810` `with bind_telemetry_stage("semantic_execute"):`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1846` `with bind_telemetry_stage("semantic_execute"):`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1925` `telemetry_id: str | None = None,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1958` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1985` `telemetry_id: str | None = None,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:1989` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:2001` `telemetry_id=(`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:2002` `get_current_telemetry().telemetry_id if telemetry_id is None else telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:2028` `telemetry_id: str | None = None,`
- `sources/openviking/openviking/storage/queuefs/semantic_processor.py:2049` `telemetry_id=telemetry_id,`

- `sources/openviking/openviking/storage/queuefs/semantic_work.py:19` `from openviking.telemetry import (`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:20` `OperationTelemetry,`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:21` `register_telemetry,`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:22` `resolve_telemetry,`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:23` `unregister_telemetry,`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:50` `def resolve_telemetry(self) -> OperationTelemetry | None:`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:51` `return resolve_telemetry(self.msg.telemetry_id)`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:140` `self._owns_telemetry = False`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:146` `if not msg.telemetry_id:`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:147` `msg.telemetry_id = OperationTelemetry(`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:149` `).telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:151` `def resolve_telemetry(self) -> OperationTelemetry:`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:152` `collector = super().resolve_telemetry()`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:154` `collector = OperationTelemetry(operation="skill_index", enabled=False)`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:155` `collector.telemetry_id = self.msg.telemetry_id`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:156` `register_telemetry(collector)`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:157` `self._owns_telemetry = True`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:188` `self.processor._merge_request_stats(self.msg.telemetry_id, error_count=len(stats.failures))`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:230` `if self._owns_telemetry:`
- `sources/openviking/openviking/storage/queuefs/semantic_work.py:231` `unregister_telemetry(self.msg.telemetry_id)`

- `sources/openviking/openviking/storage/queuefs/session_commit_processor.py:18` `from openviking.telemetry.span_models import create_root_span_attributes`

- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:57` `url = "http://{}:{}/CreateVikingdbCollection".format(host, port)`
- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:86` `url = "http://{}:{}/ListVikingdbCollection".format(host, port)`
- `sources/openviking/openviking/storage/vectordb/collection/http_collection.py:111` `self.url_prefix = "http://{}:{}/".format(ip, port)`

- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_clients.py:57` `host: VikingDB service host (e.g., "http://localhost:8080")`
- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_clients.py:70` `self._session.mount("https://", adapter)`
- `sources/openviking/openviking/storage/vectordb/collection/vikingdb_clients.py:71` `self._session.mount("http://", adapter)`

- `sources/openviking/openviking/storage/vectordb/collection/volcengine_clients.py:32` `session.mount("https://", adapter)`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_clients.py:33` `session.mount("http://", adapter)`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_clients.py:96` `url=f"https://{self.host}{req.path}",`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_clients.py:165` `url=f"https://{self.host}{req.path}",`
- `sources/openviking/openviking/storage/vectordb/collection/volcengine_clients.py:218` `url=f"https://{self.host}{req.path}",`

- `sources/openviking/openviking/storage/vectordb/engine/__init__.py:116` `requested = _normalize_requested_variant(os.environ.get("OV_ENGINE_VARIANT"))`

- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:96` `class CuVSSearchTelemetry:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:810` `telemetry: Optional[CuVSSearchTelemetry]`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1270` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1281` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1282` `telemetry.filter_kind = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1287` `telemetry.records_generation = self._records_generation`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1288` `telemetry.index_size = len(self._records)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1296` `telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1310` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1311` `telemetry.preflight_ms += (time.perf_counter() - started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1318` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1327` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1328` `telemetry.filter_kind = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1333` `telemetry.records_generation = generation`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1334` `telemetry.index_size = len(self._records)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1337` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1338` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1339` `telemetry.eligible_count = cached.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1340` `telemetry.filter_words_packed = cached.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1348` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1349` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1350` `telemetry.eligible_count = resolved.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1351` `telemetry.filter_words_packed = resolved.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1373` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1374` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1375` `telemetry.eligible_count = cached.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1376` `telemetry.filter_words_packed = cached.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1384` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1385` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1386` `telemetry.eligible_count = resolved.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1387` `telemetry.filter_words_packed = resolved.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1416` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1417` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1418` `telemetry.eligible_count = flight.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1419` `telemetry.filter_words_packed = bool(`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1469` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1470` `telemetry.filter_cache_hit = cache_hit`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1471` `telemetry.eligible_count = eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1472` `telemetry.filter_words_packed = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1609` `telemetry: Optional[CuVSSearchTelemetry],`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1622` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1623` `telemetry.memory_estimated_peak_bytes = estimate.estimated_peak_bytes`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1629` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1630` `telemetry.memory_free_bytes = free_bytes`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1631` `telemetry.memory_usable_bytes = usable_bytes`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1641` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1671` `self._check_auto_memory_budget(vector_count, telemetry)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1691` `self._check_auto_memory_budget(len(labels), telemetry)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1693` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1694` `telemetry.build_performed = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1713` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1714` `telemetry.build_ms += (time.perf_counter() - build_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1773` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1776` `candidate = self.prepare_rebuild(telemetry)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1790` `telemetry: Optional[CuVSSearchTelemetry],`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1799` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1829` `telemetry: Optional[CuVSSearchTelemetry],`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1878` `telemetry is not None`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1882` `telemetry.filter_cache_hit = False`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1883` `telemetry.filter_cache_eviction_fallback = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1894` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1895` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1896` `telemetry.eligible_count = cached_filter.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1897` `telemetry.filter_words_packed = cached_filter.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1899` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1900` `telemetry.filter_kind = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1907` `telemetry.records_generation = generation`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1908` `telemetry.index_size = len(self._records)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1916` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1922` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:1923` `telemetry.micro_batching_warm_fast_path = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2088` `if request.telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2089` `request.telemetry.micro_batching_enabled = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2090` `request.telemetry.batch_size = len(runnable)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2091` `request.telemetry.batch_wait_ms += request_wait_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2092` `request.telemetry.queue_ms += request_wait_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2125` `if request.telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2126` `request.telemetry.gpu_search_ms += gpu_search_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2177` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2195` `telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2198` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2199` `telemetry.filter_prepare_ms += (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2234` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2242` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2243` `telemetry.queue_ms += waited_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2244` `telemetry.gpu_gate_queue_ms += waited_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2254` `telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2270` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2273` `telemetry.queue_ms += waited_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2274` `telemetry.gpu_gate_queue_ms += waited_ms`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2284` `telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2299` `telemetry: Optional[CuVSSearchTelemetry] = None,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2312` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2313` `telemetry.filter_kind = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2320` `telemetry.records_generation = self._records_generation`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2321` `telemetry.index_size = len(self._records)`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2335` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2336` `telemetry.filter_cache_hit = False`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2337` `telemetry.filter_cache_eviction_fallback = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2352` `if cached_filter is not None and telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2353` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2354` `telemetry.eligible_count = cached_filter.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2355` `telemetry.filter_words_packed = cached_filter.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2358` `if resolved_native_filter is not None and telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2359` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2360` `telemetry.eligible_count = resolved_native_filter.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2361` `telemetry.filter_words_packed = resolved_native_filter.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2374` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2375` `telemetry.filter_words_packed = (`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2404` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2415` `if cached_filter is not None and telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2416` `telemetry.filter_cache_hit = True`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2417` `telemetry.filter_words_packed = cached_filter.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2424` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2425` `telemetry.filter_prepare_ms += (time.perf_counter() - filter_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2426` `telemetry.eligible_count = cached_filter.eligible_count`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2427` `telemetry.filter_words_packed = cached_filter.filter_words_packed`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2456` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2479` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/cuvs_index.py:2480` `telemetry.gpu_search_ms += (time.perf_counter() - gpu_started) * 1000.0`

- `sources/openviking/openviking/storage/vectordb/index/local_index.py:19` `CuVSSearchTelemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:886` `cuvs_telemetry: Optional[CuVSSearchTelemetry] = None`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:887` `telemetry_started = 0.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:889` `if self.dense_search and query_vector and self._cuvs_telemetry_enabled():`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:890` `cuvs_telemetry = CuVSSearchTelemetry(`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:896` `telemetry_started = time.perf_counter()`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:907` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:908` `cuvs_telemetry.route_reason = "native_rebuild_pending"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:916` `cuvs_telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:921` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:922` `cuvs_telemetry.queue_ms += (`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:929` `telemetry=cuvs_telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:932` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:933` `cuvs_telemetry.route_reason = "empty_filter"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:936` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:937` `cuvs_telemetry.route_reason = "native_filter_threshold"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:953` `cuvs_telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:960` `cuvs_telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:962` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:963` `cuvs_telemetry.route_reason = "cuvs"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:966` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:967` `cuvs_telemetry.route_reason = "native_memory_budget"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:972` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:973` `cuvs_telemetry.route_reason = "native_rebuild_pending"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:975` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:976` `cuvs_telemetry.route_reason = "native_filter_threshold"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:980` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:981` `cuvs_telemetry.route_reason = "native_unsupported_filter"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:986` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:987` `cuvs_telemetry.route_reason = "unsupported_sparse_hybrid"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:992` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:993` `cuvs_telemetry.route_reason = "native_sparse_hybrid"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1002` `cuvs_telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1005` `if cuvs_telemetry is not None and cuvs_telemetry.route_reason == "pending":`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1006` `cuvs_telemetry.route_reason = "cuvs_error"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1009` `if cuvs_telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1010` `cuvs_telemetry.total_ms += (time.perf_counter() - telemetry_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1011` `self._record_cuvs_telemetry(cuvs_telemetry)`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1019` `telemetry: Optional[CuVSSearchTelemetry],`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1031` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1032` `telemetry.queue_ms += (time.perf_counter() - queue_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1043` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1048` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1049` `telemetry.queue_ms += (time.perf_counter() - queue_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1057` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1062` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1063` `telemetry.queue_ms += (time.perf_counter() - queue_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1071` `telemetry=telemetry,`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1082` `telemetry: Optional[CuVSSearchTelemetry],`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1095` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1096` `telemetry.native_filter_reused = True`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1106` `if telemetry is not None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1107` `telemetry.native_search_ms += (time.perf_counter() - native_started) * 1000.0`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1108` `if telemetry.route_reason == "pending":`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1109` `telemetry.route_reason = "native_fallback"`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1132` `def _cuvs_telemetry_enabled() -> bool:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1136` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1138` `return bool(get_current_telemetry().enabled)`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1143` `def _record_cuvs_telemetry(telemetry: CuVSSearchTelemetry) -> None:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1144` `payload = telemetry.as_dict()`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1146` `logger.debug("cuVS search telemetry: %s", json.dumps(payload, sort_keys=True))`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1148` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1150` `operation_telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1151` `if not operation_telemetry.enabled:`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1153` `operation_telemetry.record_cuvs_search(payload)`
- `sources/openviking/openviking/storage/vectordb/index/local_index.py:1155` `logger.debug("Failed to record cuVS search telemetry", exc_info=True)`

- `sources/openviking/openviking/storage/vectordb/service/api_fastapi.py:69` `PERSIST_PATH = os.environ.get("VIKINGDB_PERSIST_PATH", "./vikingdb_data/")`

- `sources/openviking/openviking/storage/vectordb/utils/config_utils.py:27` `env_value = os.environ.get(env_var)`

- `sources/openviking/openviking/storage/vectordb_adapters/base.py:63` `if not normalized.startswith(("http://", "https://")):`
- `sources/openviking/openviking/storage/vectordb_adapters/base.py:64` `normalized = f"http://{normalized}"`

- `sources/openviking/openviking/storage/viking_fs/__init__.py:85` `from openviking.telemetry import get_current_telemetry`

- `sources/openviking/openviking/storage/viking_fs/_base.py:178` `env_val = os.getenv("OPENVIKING_FILE_OPS_CONCURRENCY")`

- `sources/openviking/openviking/storage/viking_fs/_semantic.py:27` `from openviking.telemetry import get_current_telemetry`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:247` `telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:332` `telemetry.set("vector.returned", find_result.total)`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:357` `telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:392` `telemetry.set("vector.returned", find_result.total)`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:430` `telemetry = get_current_telemetry()`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:456` `with telemetry.measure("search.target_abstract"):`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:496` `with telemetry.measure("search.intent_analysis"):`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:525` `telemetry.set("search.typed_queries_count", len(typed_queries))`
- `sources/openviking/openviking/storage/viking_fs/_semantic.py:574` `telemetry.set("vector.returned", find_result.total)`

- `sources/openviking/openviking/telemetry/__init__.py:3` `"""OpenViking telemetry runtime and operation telemetry helpers."""`
- `sources/openviking/openviking/telemetry/__init__.py:8` `bind_telemetry,`
- `sources/openviking/openviking/telemetry/__init__.py:9` `bind_telemetry_stage,`
- `sources/openviking/openviking/telemetry/__init__.py:10` `get_current_telemetry,`
- `sources/openviking/openviking/telemetry/__init__.py:11` `get_current_telemetry_stage,`
- `sources/openviking/openviking/telemetry/__init__.py:14` `from .operation import OperationTelemetry, TelemetrySnapshot`
- `sources/openviking/openviking/telemetry/__init__.py:15` `from .registry import register_telemetry, resolve_telemetry, unregister_telemetry`
- `sources/openviking/openviking/telemetry/__init__.py:16` `from .request import TelemetryRequest, TelemetrySelection, normalize_telemetry_request`
- `sources/openviking/openviking/telemetry/__init__.py:17` `from .runtime import get_telemetry_runtime, set_telemetry_runtime`
- `sources/openviking/openviking/telemetry/__init__.py:21` `"OperationTelemetry",`
- `sources/openviking/openviking/telemetry/__init__.py:22` `"TelemetryRequest",`
- `sources/openviking/openviking/telemetry/__init__.py:23` `"TelemetrySelection",`
- `sources/openviking/openviking/telemetry/__init__.py:24` `"TelemetrySnapshot",`
- `sources/openviking/openviking/telemetry/__init__.py:26` `"bind_telemetry",`
- `sources/openviking/openviking/telemetry/__init__.py:27` `"bind_telemetry_stage",`
- `sources/openviking/openviking/telemetry/__init__.py:28` `"get_current_telemetry",`
- `sources/openviking/openviking/telemetry/__init__.py:29` `"get_current_telemetry_stage",`
- `sources/openviking/openviking/telemetry/__init__.py:31` `"get_telemetry_runtime",`
- `sources/openviking/openviking/telemetry/__init__.py:32` `"normalize_telemetry_request",`
- `sources/openviking/openviking/telemetry/__init__.py:33` `"register_telemetry",`
- `sources/openviking/openviking/telemetry/__init__.py:34` `"resolve_telemetry",`
- `sources/openviking/openviking/telemetry/__init__.py:35` `"set_telemetry_runtime",`
- `sources/openviking/openviking/telemetry/__init__.py:39` `"unregister_telemetry",`

- `sources/openviking/openviking/telemetry/backends/__init__.py:3` `"""Telemetry backend exports."""`
- `sources/openviking/openviking/telemetry/backends/__init__.py:5` `from .memory import MemoryOperationTelemetry`
- `sources/openviking/openviking/telemetry/backends/__init__.py:7` `__all__ = ["MemoryOperationTelemetry"]`

- `sources/openviking/openviking/telemetry/backends/memory.py:3` `"""Memory telemetry backend."""`
- `sources/openviking/openviking/telemetry/backends/memory.py:5` `from openviking.telemetry.operation import OperationTelemetry`
- `sources/openviking/openviking/telemetry/backends/memory.py:8` `class MemoryOperationTelemetry(OperationTelemetry):`
- `sources/openviking/openviking/telemetry/backends/memory.py:9` `"""In-process operation telemetry collector."""`
- `sources/openviking/openviking/telemetry/backends/memory.py:12` `__all__ = ["MemoryOperationTelemetry"]`

- `sources/openviking/openviking/telemetry/context.py:3` `"""Telemetry context helpers."""`
- `sources/openviking/openviking/telemetry/context.py:11` `from .operation import OperationTelemetry`
- `sources/openviking/openviking/telemetry/context.py:13` `_CURRENT_TELEMETRY: contextvars.ContextVar[OperationTelemetry | None] = contextvars.ContextVar(`
- `sources/openviking/openviking/telemetry/context.py:14` `"openviking_operation_telemetry",`
- `sources/openviking/openviking/telemetry/context.py:17` `_CURRENT_TELEMETRY_STAGE: contextvars.ContextVar[str | None] = contextvars.ContextVar(`
- `sources/openviking/openviking/telemetry/context.py:18` `"openviking_operation_telemetry_stage",`
- `sources/openviking/openviking/telemetry/context.py:27` `def get_current_telemetry() -> OperationTelemetry:`
- `sources/openviking/openviking/telemetry/context.py:28` `"""Get current operation telemetry or create a request-local disabled collector."""`
- `sources/openviking/openviking/telemetry/context.py:29` `telemetry = _CURRENT_TELEMETRY.get()`
- `sources/openviking/openviking/telemetry/context.py:30` `if telemetry is None:`
- `sources/openviking/openviking/telemetry/context.py:31` `telemetry = OperationTelemetry(operation="noop", enabled=False)`
- `sources/openviking/openviking/telemetry/context.py:32` `_CURRENT_TELEMETRY.set(telemetry)`
- `sources/openviking/openviking/telemetry/context.py:33` `return telemetry`
- `sources/openviking/openviking/telemetry/context.py:37` `def bind_telemetry(handle: OperationTelemetry) -> Iterator[OperationTelemetry]:`
- `sources/openviking/openviking/telemetry/context.py:38` `"""Bind operation telemetry to current context."""`
- `sources/openviking/openviking/telemetry/context.py:39` `token = _CURRENT_TELEMETRY.set(handle)`
- `sources/openviking/openviking/telemetry/context.py:43` `_CURRENT_TELEMETRY.reset(token)`
- `sources/openviking/openviking/telemetry/context.py:46` `def get_current_telemetry_stage() -> str | None:`
- `sources/openviking/openviking/telemetry/context.py:47` `"""Return the currently bound low-cardinality telemetry stage, if any."""`
- `sources/openviking/openviking/telemetry/context.py:48` `stage = _CURRENT_TELEMETRY_STAGE.get()`
- `sources/openviking/openviking/telemetry/context.py:56` `def bind_telemetry_stage(stage: str | None) -> Iterator[str | None]:`
- `sources/openviking/openviking/telemetry/context.py:59` `token = _CURRENT_TELEMETRY_STAGE.set(normalized)`
- `sources/openviking/openviking/telemetry/context.py:63` `_CURRENT_TELEMETRY_STAGE.reset(token)`
- `sources/openviking/openviking/telemetry/context.py:83` `"bind_telemetry",`
- `sources/openviking/openviking/telemetry/context.py:84` `"bind_telemetry_stage",`
- `sources/openviking/openviking/telemetry/context.py:85` `"get_current_telemetry",`
- `sources/openviking/openviking/telemetry/context.py:86` `"get_current_telemetry_stage",`

- `sources/openviking/openviking/telemetry/execution.py:3` `"""Shared helpers for telemetry-wrapped operation execution."""`
- `sources/openviking/openviking/telemetry/execution.py:17` `from .context import bind_telemetry`
- `sources/openviking/openviking/telemetry/execution.py:18` `from .operation import OperationTelemetry, TelemetrySnapshot`
- `sources/openviking/openviking/telemetry/execution.py:19` `from .request import TelemetryRequest, TelemetrySelection, normalize_telemetry_request`
- `sources/openviking/openviking/telemetry/execution.py:27` `class TelemetryExecutionResult(Generic[T]):`
- `sources/openviking/openviking/telemetry/execution.py:28` `"""Executed operation result plus telemetry payloads."""`
- `sources/openviking/openviking/telemetry/execution.py:31` `telemetry: Optional[dict[str, Any]]`
- `sources/openviking/openviking/telemetry/execution.py:32` `selection: TelemetrySelection`
- `sources/openviking/openviking/telemetry/execution.py:35` `def parse_telemetry_selection(telemetry: TelemetryRequest) -> TelemetrySelection:`
- `sources/openviking/openviking/telemetry/execution.py:36` `"""Validate and normalize a telemetry request for public API usage."""`
- `sources/openviking/openviking/telemetry/execution.py:38` `return normalize_telemetry_request(telemetry)`
- `sources/openviking/openviking/telemetry/execution.py:43` `def build_telemetry_payload(`
- `sources/openviking/openviking/telemetry/execution.py:44` `snapshot: Optional[TelemetrySnapshot],`
- `sources/openviking/openviking/telemetry/execution.py:45` `selection: TelemetrySelection,`
- `sources/openviking/openviking/telemetry/execution.py:47` `"""Build a telemetry payload from a finished snapshot."""`
- `sources/openviking/openviking/telemetry/execution.py:53` `def _log_telemetry_summary(snapshot: Optional[TelemetrySnapshot]) -> None:`
- `sources/openviking/openviking/telemetry/execution.py:57` `"Telemetry summary (id=%s): %s",`
- `sources/openviking/openviking/telemetry/execution.py:58` `snapshot.telemetry_id,`
- `sources/openviking/openviking/telemetry/execution.py:63` `def attach_telemetry_payload(`
- `sources/openviking/openviking/telemetry/execution.py:65` `telemetry_payload: Optional[dict[str, Any]],`
- `sources/openviking/openviking/telemetry/execution.py:67` `"""Attach a telemetry payload to a dict result."""`
- `sources/openviking/openviking/telemetry/execution.py:68` `if telemetry_payload is None:`
- `sources/openviking/openviking/telemetry/execution.py:73` `payload["telemetry"] = telemetry_payload`
- `sources/openviking/openviking/telemetry/execution.py:77` `result["telemetry"] = telemetry_payload`
- `sources/openviking/openviking/telemetry/execution.py:85` `from opentelemetry import trace as otel_trace`
- `sources/openviking/openviking/telemetry/execution.py:86` `from opentelemetry.trace import Status, StatusCode`
- `sources/openviking/openviking/telemetry/execution.py:93` `def _fill_telemetry_summary_to_span(snapshot, span):`
- `sources/openviking/openviking/telemetry/execution.py:94` `"""fill OperationTelemetry summary into span`
- `sources/openviking/openviking/telemetry/execution.py:97` `snapshot: TelemetrySnapshot object`
- `sources/openviking/openviking/telemetry/execution.py:105` `op_attrs = OperationSpanAttributes.from_telemetry_snapshot(`
- `sources/openviking/openviking/telemetry/execution.py:107` `telemetry_id=snapshot.telemetry_id,`
- `sources/openviking/openviking/telemetry/execution.py:117` `span.add_event("telemetry_summary", attributes={"summary": str(snapshot.summary)})`
- `sources/openviking/openviking/telemetry/execution.py:119` `logger.debug("failed to fill telemetry summary into span", exc_info=True)`
- `sources/openviking/openviking/telemetry/execution.py:123` `async def run_with_telemetry(`
- `sources/openviking/openviking/telemetry/execution.py:126` `telemetry: TelemetryRequest,`
- `sources/openviking/openviking/telemetry/execution.py:129` `) -> TelemetryExecutionResult[T]:`
- `sources/openviking/openviking/telemetry/execution.py:131` `selection = parse_telemetry_selection(telemetry)`
- `sources/openviking/openviking/telemetry/execution.py:132` `collector = OperationTelemetry(`
- `sources/openviking/openviking/telemetry/execution.py:138` `telemetry_id=collector.telemetry_id,`
- `sources/openviking/openviking/telemetry/execution.py:157` `with bind_telemetry(collector):`
- `sources/openviking/openviking/telemetry/execution.py:173` `_log_telemetry_summary(snapshot)`
- `sources/openviking/openviking/telemetry/execution.py:182` `_fill_telemetry_summary_to_span(snapshot, current_span)`
- `sources/openviking/openviking/telemetry/execution.py:192` `_log_telemetry_summary(snapshot)`
- `sources/openviking/openviking/telemetry/execution.py:199` `_fill_telemetry_summary_to_span(snapshot, current_span)`
- `sources/openviking/openviking/telemetry/execution.py:201` `logger.debug("failed to backfill telemetry summary into span", exc_info=True)`
- `sources/openviking/openviking/telemetry/execution.py:205` `from openviking.metrics.datasources.telemetry_bridge import (`
- `sources/openviking/openviking/telemetry/execution.py:206` `TelemetryBridgeEventDataSource,`
- `sources/openviking/openviking/telemetry/execution.py:209` `TelemetryBridgeEventDataSource.record_summary(snapshot.summary)`
- `sources/openviking/openviking/telemetry/execution.py:211` `logger.debug("failed to record telemetry summary into metrics bridge", exc_info=True)`
- `sources/openviking/openviking/telemetry/execution.py:212` `telemetry_payload = build_telemetry_payload(`
- `sources/openviking/openviking/telemetry/execution.py:216` `return TelemetryExecutionResult(`
- `sources/openviking/openviking/telemetry/execution.py:218` `telemetry=telemetry_payload,`
- `sources/openviking/openviking/telemetry/execution.py:226` `"TelemetryExecutionResult",`
- `sources/openviking/openviking/telemetry/execution.py:227` `"attach_telemetry_payload",`
- `sources/openviking/openviking/telemetry/execution.py:228` `"build_telemetry_payload",`
- `sources/openviking/openviking/telemetry/execution.py:229` `"parse_telemetry_selection",`
- `sources/openviking/openviking/telemetry/execution.py:230` `"run_with_telemetry",`

- `sources/openviking/openviking/telemetry/operation.py:3` `"""Operation-scoped telemetry primitives."""`
- `sources/openviking/openviking/telemetry/operation.py:45` `class TelemetrySnapshot:`
- `sources/openviking/openviking/telemetry/operation.py:46` `"""Final operation telemetry output."""`
- `sources/openviking/openviking/telemetry/operation.py:48` `telemetry_id: str`
- `sources/openviking/openviking/telemetry/operation.py:56` `payload: Dict[str, Any] = {"id": self.telemetry_id}`
- `sources/openviking/openviking/telemetry/operation.py:62` `class TelemetrySummaryBuilder:`
- `sources/openviking/openviking/telemetry/operation.py:523` `class OperationTelemetry:`
- `sources/openviking/openviking/telemetry/operation.py:524` `"""Operation-scoped telemetry collector with low-overhead disabled mode."""`
- `sources/openviking/openviking/telemetry/operation.py:533` `self.telemetry_id = f"tm_{uuid4().hex}"`
- `sources/openviking/openviking/telemetry/operation.py:600` `if TelemetrySummaryBuilder._bool(metrics.get("auto_mode"), False):`
- `sources/openviking/openviking/telemetry/operation.py:602` `if TelemetrySummaryBuilder._bool(metrics.get("micro_batching_enabled"), False):`
- `sources/openviking/openviking/telemetry/operation.py:604` `max(TelemetrySummaryBuilder._i(metrics.get("batch_size"), 1), 1),`
- `sources/openviking/openviking/telemetry/operation.py:612` `TelemetrySummaryBuilder._i(self._gauges.get("vector.cuvs.batch_size.max"), 1),`
- `sources/openviking/openviking/telemetry/operation.py:615` `if TelemetrySummaryBuilder._bool(metrics.get("micro_batching_warm_fast_path"), False):`
- `sources/openviking/openviking/telemetry/operation.py:617` `if TelemetrySummaryBuilder._bool(metrics.get("filter_cache_hit"), False):`
- `sources/openviking/openviking/telemetry/operation.py:619` `if TelemetrySummaryBuilder._bool(metrics.get("filter_cache_eviction_fallback"), False):`
- `sources/openviking/openviking/telemetry/operation.py:621` `if TelemetrySummaryBuilder._bool(metrics.get("filter_words_packed"), False):`
- `sources/openviking/openviking/telemetry/operation.py:623` `if TelemetrySummaryBuilder._bool(metrics.get("native_filter_reused"), False):`
- `sources/openviking/openviking/telemetry/operation.py:625` `if TelemetrySummaryBuilder._bool(metrics.get("build_performed"), False):`
- `sources/openviking/openviking/telemetry/operation.py:629` `TelemetrySummaryBuilder._i(metrics.get("max_concurrent_gpu_searches"), 1), 1`
- `sources/openviking/openviking/telemetry/operation.py:632` `TelemetrySummaryBuilder._i(`
- `sources/openviking/openviking/telemetry/operation.py:641` `value = max(TelemetrySummaryBuilder._i(raw_value, 0), 0)`
- `sources/openviking/openviking/telemetry/operation.py:643` `self._gauges[key] = max(TelemetrySummaryBuilder._i(self._gauges.get(key), 0), value)`
- `sources/openviking/openviking/telemetry/operation.py:646` `peak = TelemetrySummaryBuilder._i(peak_raw, -1)`
- `sources/openviking/openviking/telemetry/operation.py:649` `self._gauges[key] = max(TelemetrySummaryBuilder._i(self._gauges.get(key), 0), peak)`
- `sources/openviking/openviking/telemetry/operation.py:652` `value = TelemetrySummaryBuilder._i(raw_value, -1)`
- `sources/openviking/openviking/telemetry/operation.py:658` `value if existing is None else min(TelemetrySummaryBuilder._i(existing), value)`
- `sources/openviking/openviking/telemetry/operation.py:666` `TelemetrySummaryBuilder._i(self._counters.get(sum_key), 0) + value_us`
- `sources/openviking/openviking/telemetry/operation.py:669` `TelemetrySummaryBuilder._i(self._gauges.get(max_key), 0), value_us`
- `sources/openviking/openviking/telemetry/operation.py:773` `from .context import get_current_telemetry_stage`
- `sources/openviking/openviking/telemetry/operation.py:775` `stage = get_current_telemetry_stage()`
- `sources/openviking/openviking/telemetry/operation.py:805` `def finish(self, status: str = "ok") -> Optional[TelemetrySnapshot]:`
- `sources/openviking/openviking/telemetry/operation.py:811` `summary = TelemetrySummaryBuilder.build(`
- `sources/openviking/openviking/telemetry/operation.py:821` `return TelemetrySnapshot(`
- `sources/openviking/openviking/telemetry/operation.py:822` `telemetry_id=self.telemetry_id,`
- `sources/openviking/openviking/telemetry/operation.py:828` `"OperationTelemetry",`
- `sources/openviking/openviking/telemetry/operation.py:829` `"TelemetrySnapshot",`
- `sources/openviking/openviking/telemetry/operation.py:830` `"TelemetrySummaryBuilder",`

- `sources/openviking/openviking/telemetry/registry.py:3` `"""Telemetry registry helpers."""`
- `sources/openviking/openviking/telemetry/registry.py:9` `from .operation import OperationTelemetry`
- `sources/openviking/openviking/telemetry/registry.py:11` `_REGISTERED_TELEMETRY: dict[str, OperationTelemetry] = {}`
- `sources/openviking/openviking/telemetry/registry.py:12` `_REGISTERED_TELEMETRY_LOCK = threading.Lock()`
- `sources/openviking/openviking/telemetry/registry.py:15` `def register_telemetry(handle: OperationTelemetry) -> None:`
- `sources/openviking/openviking/telemetry/registry.py:16` `if not handle.enabled or not handle.telemetry_id:`
- `sources/openviking/openviking/telemetry/registry.py:18` `with _REGISTERED_TELEMETRY_LOCK:`
- `sources/openviking/openviking/telemetry/registry.py:19` `_REGISTERED_TELEMETRY[handle.telemetry_id] = handle`
- `sources/openviking/openviking/telemetry/registry.py:22` `def resolve_telemetry(telemetry_id: str) -> OperationTelemetry | None:`
- `sources/openviking/openviking/telemetry/registry.py:23` `if not telemetry_id:`
- `sources/openviking/openviking/telemetry/registry.py:25` `with _REGISTERED_TELEMETRY_LOCK:`
- `sources/openviking/openviking/telemetry/registry.py:26` `return _REGISTERED_TELEMETRY.get(telemetry_id)`
- `sources/openviking/openviking/telemetry/registry.py:29` `def unregister_telemetry(telemetry_id: str) -> None:`
- `sources/openviking/openviking/telemetry/registry.py:30` `if not telemetry_id:`
- `sources/openviking/openviking/telemetry/registry.py:32` `with _REGISTERED_TELEMETRY_LOCK:`
- `sources/openviking/openviking/telemetry/registry.py:33` `_REGISTERED_TELEMETRY.pop(telemetry_id, None)`
- `sources/openviking/openviking/telemetry/registry.py:36` `__all__ = ["register_telemetry", "resolve_telemetry", "unregister_telemetry"]`

- `sources/openviking/openviking/telemetry/request.py:3` `"""Telemetry request parsing helpers."""`
- `sources/openviking/openviking/telemetry/request.py:11` `TelemetryRequest: TypeAlias = bool | dict[str, bool]`
- `sources/openviking/openviking/telemetry/request.py:13` `_ALLOWED_TELEMETRY_KEYS = frozenset({"summary"})`
- `sources/openviking/openviking/telemetry/request.py:17` `class TelemetrySelection:`
- `sources/openviking/openviking/telemetry/request.py:18` `"""Normalized telemetry payload selection."""`
- `sources/openviking/openviking/telemetry/request.py:27` `def normalize_telemetry_request(`
- `sources/openviking/openviking/telemetry/request.py:28` `request: TelemetryRequest | Mapping[str, Any] | None,`
- `sources/openviking/openviking/telemetry/request.py:29` `) -> TelemetrySelection:`
- `sources/openviking/openviking/telemetry/request.py:30` `"""Normalize a telemetry request into explicit response selection flags."""`
- `sources/openviking/openviking/telemetry/request.py:32` `return TelemetrySelection(include_summary=False)`
- `sources/openviking/openviking/telemetry/request.py:34` `return TelemetrySelection(include_summary=True)`
- `sources/openviking/openviking/telemetry/request.py:36` `raise ValueError("telemetry must be a boolean or an object")`
- `sources/openviking/openviking/telemetry/request.py:38` `unknown_keys = set(request) - _ALLOWED_TELEMETRY_KEYS`
- `sources/openviking/openviking/telemetry/request.py:41` `raise ValueError(f"Unsupported telemetry options: {joined}")`
- `sources/openviking/openviking/telemetry/request.py:45` `raise ValueError("telemetry.summary must be a boolean")`
- `sources/openviking/openviking/telemetry/request.py:46` `return TelemetrySelection(include_summary=include_summary)`
- `sources/openviking/openviking/telemetry/request.py:49` `__all__ = ["TelemetryRequest", "TelemetrySelection", "normalize_telemetry_request"]`

- `sources/openviking/openviking/telemetry/resource_summary.py:3` `"""Resource-specific telemetry summary helpers."""`
- `sources/openviking/openviking/telemetry/resource_summary.py:9` `from .operation import OperationTelemetry`
- `sources/openviking/openviking/telemetry/resource_summary.py:12` `def _consume_semantic_request_stats(telemetry_id: str):`
- `sources/openviking/openviking/telemetry/resource_summary.py:16` `return SemanticProcessor.consume_request_stats(telemetry_id)`
- `sources/openviking/openviking/telemetry/resource_summary.py:21` `def _consume_embedding_request_stats(telemetry_id: str):`
- `sources/openviking/openviking/telemetry/resource_summary.py:25` `return TextEmbeddingHandler.consume_request_stats(telemetry_id)`
- `sources/openviking/openviking/telemetry/resource_summary.py:30` `def _consume_semantic_tree_stats(telemetry_id: str, root_uri: str | None):`
- `sources/openviking/openviking/telemetry/resource_summary.py:34` `return SemanticProcessor.consume_tree_stats(telemetry_id=telemetry_id, uri=root_uri)`
- `sources/openviking/openviking/telemetry/resource_summary.py:51` `telemetry: OperationTelemetry,`
- `sources/openviking/openviking/telemetry/resource_summary.py:52` `telemetry_id: str,`
- `sources/openviking/openviking/telemetry/resource_summary.py:57` `if not telemetry.enabled:`
- `sources/openviking/openviking/telemetry/resource_summary.py:60` `semantic = _queue_metrics(_consume_semantic_request_stats(telemetry_id))`
- `sources/openviking/openviking/telemetry/resource_summary.py:61` `embedding = _queue_metrics(_consume_embedding_request_stats(telemetry_id))`
- `sources/openviking/openviking/telemetry/resource_summary.py:66` `telemetry.set("queue.semantic.processed", semantic["processed"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:67` `telemetry.set("queue.semantic.requeue_count", semantic["requeue_count"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:68` `telemetry.set("queue.semantic.error_count", semantic["error_count"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:69` `telemetry.set("queue.semantic.queue_wait.duration_ms", semantic_timing["queue_wait_ms"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:70` `telemetry.set("queue.semantic.execute.duration_ms", semantic_timing["execute_ms"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:71` `telemetry.set("queue.embedding.processed", embedding["processed"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:72` `telemetry.set("queue.embedding.requeue_count", embedding["requeue_count"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:73` `telemetry.set("queue.embedding.error_count", embedding["error_count"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:74` `telemetry.set("queue.embedding.queue_wait.duration_ms", embedding_timing["queue_wait_ms"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:75` `telemetry.set("queue.embedding.execute.duration_ms", embedding_timing["execute_ms"])`
- `sources/openviking/openviking/telemetry/resource_summary.py:77` `tree_stats = _consume_semantic_tree_stats(telemetry_id, root_uri)`
- `sources/openviking/openviking/telemetry/resource_summary.py:79` `telemetry.set("semantic_nodes.total", tree_stats.total_nodes)`
- `sources/openviking/openviking/telemetry/resource_summary.py:80` `telemetry.set("semantic_nodes.done", tree_stats.done_nodes)`
- `sources/openviking/openviking/telemetry/resource_summary.py:81` `telemetry.set("semantic_nodes.pending", tree_stats.pending_nodes)`
- `sources/openviking/openviking/telemetry/resource_summary.py:82` `telemetry.set("semantic_nodes.running", tree_stats.in_progress_nodes)`

- `sources/openviking/openviking/telemetry/runtime.py:3` `"""Telemetry runtime entrypoints."""`
- `sources/openviking/openviking/telemetry/runtime.py:12` `class MemoryTelemetryMeter:`
- `sources/openviking/openviking/telemetry/runtime.py:13` `"""Lightweight in-process telemetry meter.`
- `sources/openviking/openviking/telemetry/runtime.py:15` `This is intentionally simple for now; it provides the global telemetry hook`
- `sources/openviking/openviking/telemetry/runtime.py:55` `class TelemetryRuntime:`
- `sources/openviking/openviking/telemetry/runtime.py:56` `meter_instance: MemoryTelemetryMeter = field(default_factory=MemoryTelemetryMeter)`
- `sources/openviking/openviking/telemetry/runtime.py:58` `_RUNTIME = TelemetryRuntime()`
- `sources/openviking/openviking/telemetry/runtime.py:61` `def get_telemetry_runtime() -> TelemetryRuntime:`
- `sources/openviking/openviking/telemetry/runtime.py:65` `def set_telemetry_runtime(runtime: TelemetryRuntime) -> None:`
- `sources/openviking/openviking/telemetry/runtime.py:71` `"MemoryTelemetryMeter",`
- `sources/openviking/openviking/telemetry/runtime.py:72` `"TelemetryRuntime",`
- `sources/openviking/openviking/telemetry/runtime.py:73` `"get_telemetry_runtime",`
- `sources/openviking/openviking/telemetry/runtime.py:74` `"set_telemetry_runtime",`

- `sources/openviking/openviking/telemetry/snapshot.py:3` `"""Telemetry snapshot model."""`
- `sources/openviking/openviking/telemetry/snapshot.py:5` `from .operation import TelemetrySnapshot`
- `sources/openviking/openviking/telemetry/snapshot.py:7` `__all__ = ["TelemetrySnapshot"]`

- `sources/openviking/openviking/telemetry/span_models.py:16` `- Lightweight integration with OpenTelemetry attribute APIs`
- `sources/openviking/openviking/telemetry/span_models.py:70` `"""Convert the model into an OpenTelemetry span attribute mapping."""`
- `sources/openviking/openviking/telemetry/span_models.py:141` `telemetry_id: str`
- `sources/openviking/openviking/telemetry/span_models.py:142` `"""Telemetry identifier for one operation execution."""`
- `sources/openviking/openviking/telemetry/span_models.py:176` `"""Convert the model into an OpenTelemetry span attribute mapping."""`
- `sources/openviking/openviking/telemetry/span_models.py:179` `"telemetry_id": self.telemetry_id,`
- `sources/openviking/openviking/telemetry/span_models.py:207` `"telemetry_id": self.telemetry_id,`
- `sources/openviking/openviking/telemetry/span_models.py:235` `def from_telemetry_snapshot(`
- `sources/openviking/openviking/telemetry/span_models.py:238` `telemetry_id: str,`
- `sources/openviking/openviking/telemetry/span_models.py:242` `"""Build an instance from an 'OperationTelemetry' summary snapshot."""`
- `sources/openviking/openviking/telemetry/span_models.py:245` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/telemetry/span_models.py:314` `telemetry_id: str,`
- `sources/openviking/openviking/telemetry/span_models.py:320` `telemetry_id=telemetry_id,`
- `sources/openviking/openviking/telemetry/span_models.py:354` `from opentelemetry import trace as otel_trace`

- `sources/openviking/openviking/telemetry/tracer.py:3` `"""OpenTelemetry tracer integration for OpenViking."""`
- `sources/openviking/openviking/telemetry/tracer.py:19` `from opentelemetry import trace as otel_trace`
- `sources/openviking/openviking/telemetry/tracer.py:20` `from opentelemetry.context import Context`
- `sources/openviking/openviking/telemetry/tracer.py:21` `from opentelemetry.propagate import extract, inject`
- `sources/openviking/openviking/telemetry/tracer.py:22` `from opentelemetry.sdk.resources import Resource`
- `sources/openviking/openviking/telemetry/tracer.py:23` `from opentelemetry.sdk.trace import Status, StatusCode, TracerProvider`
- `sources/openviking/openviking/telemetry/tracer.py:24` `from opentelemetry.sdk.trace.export import BatchSpanProcessor, SpanExporter, SpanExportResult`
- `sources/openviking/openviking/telemetry/tracer.py:25` `from opentelemetry.trace.propagation.tracecontext import TraceContextTextMapPropagator`
- `sources/openviking/openviking/telemetry/tracer.py:29` `from opentelemetry.exporter.otlp.proto.common.trace_encoder import encode_spans`
- `sources/openviking/openviking/telemetry/tracer.py:35` `from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import (`
- `sources/openviking/openviking/telemetry/tracer.py:42` `from opentelemetry.exporter.otlp.proto.http.trace_exporter import (`
- `sources/openviking/openviking/telemetry/tracer.py:80` `"""OpenTelemetry span exporter that writes OTLP JSON batches to a local JSONL file.`
- `sources/openviking/openviking/telemetry/tracer.py:100` `raise ImportError("OpenTelemetry trace exporter dependencies are not available")`
- `sources/openviking/openviking/telemetry/tracer.py:259` `from opentelemetry.instrumentation.asyncio import AsyncioInstrumentor`
- `sources/openviking/openviking/telemetry/tracer.py:264` `logger.warning("[TRACER] opentelemetry-instrumentation-asyncio not installed")`
- `sources/openviking/openviking/telemetry/tracer.py:280` `"""Initialize the OpenTelemetry tracer.`
- `sources/openviking/openviking/telemetry/tracer.py:304` `"OpenTelemetry not installed. Install with: uv pip install opentelemetry-api "`
- `sources/openviking/openviking/telemetry/tracer.py:305` `"opentelemetry-sdk opentelemetry-exporter-otlpprotogrpc"`
- `sources/openviking/openviking/telemetry/tracer.py:334` `if not endpoint.startswith("http://") and not endpoint.startswith("https://"):`
- `sources/openviking/openviking/telemetry/tracer.py:336` `"OTLP/HTTP trace endpoint must include scheme, e.g. 'http://localhost:4318/v1/traces'"`

- `sources/openviking/openviking_cli/utils/config/__init__.py:90` `from .telemetry_config import TelemetryConfig, TracerConfig`
- `sources/openviking/openviking_cli/utils/config/__init__.py:178` `"TelemetryConfig",`

- `sources/openviking/openviking_cli/utils/config/agfs_config.py:123` `db_path: Optional[str] = Field(`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:371` `queue_db_path: Optional[str] = Field(`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:456` `if self.queue_db_path is not None and self.queuefs.db_path is None:`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:458` `"AGFSConfig: 'storage.agfs.queue_db_path' is deprecated; "`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:459` `"prefer 'storage.agfs.queuefs.db_path'."`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:463` `if self.queuefs.db_path is not None or self.queue_db_path is not None:`
- `sources/openviking/openviking_cli/utils/config/agfs_config.py:466` `"db_path/queue_db_path will be ignored."`

- `sources/openviking/openviking_cli/utils/config/config_loader.py:47` `env_val = os.environ.get(env_var)`
- `sources/openviking/openviking_cli/utils/config/config_loader.py:124` `f"See: https://openviking.ai/docs"`

- `sources/openviking/openviking_cli/utils/config/consts.py:7` `DEFAULT_CONFIG_DIR = Path.home() / ".openviking"`

- `sources/openviking/openviking_cli/utils/config/embedding_config.py:926` `"api_base": cfg.api_base or "http://localhost:11434/v1",`

- `sources/openviking/openviking_cli/utils/config/encryption_config.py:106` `"address": "https://vault.example.com:8200",`

- `sources/openviking/openviking_cli/utils/config/ingest_config.py:112` `enabled = _env_truthy(os.environ.get(OPENVIKING_INGEST_ENABLED_ENV))`
- `sources/openviking/openviking_cli/utils/config/ingest_config.py:115` `server_url = os.environ.get(OPENVIKING_INGEST_SERVER_URL_ENV)`
- `sources/openviking/openviking_cli/utils/config/ingest_config.py:118` `api_key = os.environ.get(OPENVIKING_INGEST_API_KEY_ENV)`

- `sources/openviking/openviking_cli/utils/config/oauth_config.py:34` `"Public issuer URL (e.g. 'https://ov.example.com'). Resolution order: the "`

- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:65` `from .telemetry_config import TelemetryConfig`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:107` `"(e.g., http://...)"`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:149` `raise ValueError("parser_api.host must include scheme (e.g., https://...)")`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:186` `raise ValueError("compile_api.base_url must include scheme (e.g., https://...)")`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:510` `telemetry: "TelemetryConfig" = Field(`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:511` `default_factory=TelemetryConfig, description="Telemetry configuration"`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:729` `f"See: https://openviking.ai/docs"`
- `sources/openviking/openviking_cli/utils/config/open_viking_config.py:769` `f"See: https://openviking.ai/docs"`

- `sources/openviking/openviking_cli/utils/config/parser_config.py:549` `domain: str = "https://open.feishu.cn"`

- `sources/openviking/openviking_cli/utils/config/telemetry_config.py:9` `"""OpenTelemetry tracer configuration."""`
- `sources/openviking/openviking_cli/utils/config/telemetry_config.py:11` `enabled: bool = Field(default=False, description="Enable OpenTelemetry tracing")`
- `sources/openviking/openviking_cli/utils/config/telemetry_config.py:19` `class TelemetryConfig(BaseModel):`
- `sources/openviking/openviking_cli/utils/config/telemetry_config.py:20` `"""Telemetry configuration including tracer."""`
- `sources/openviking/openviking_cli/utils/config/telemetry_config.py:23` `default_factory=lambda: TracerConfig(), description="OpenTelemetry tracer configuration"`

- `sources/openviking/openviking_cli/utils/config/vectordb_config.py:421` `description="Remote service URL for 'http' type (e.g., 'http://localhost:5000')",`

- `sources/openviking/openviking_gateway/config.py:16` `PUBLIC_OPENVIKING_URLS = frozenset({"https://api.vikingdb.cn-beijing.volces.com/openviking"})`
- `sources/openviking/openviking_gateway/config.py:68` `override = os.environ.get(ENVIRONMENT_PREFIX + field.upper(), "")`

- `sources/openviking/openviking_gateway/storage.py:53` `path.parent.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/openviking/openviking_gateway/storage.py:56` `fd = os.open(path, os.O_CREAT | os.O_RDWR, 0o600)`
- `sources/openviking/openviking_gateway/storage.py:57` `os.fchmod(fd, 0o600)`
- `sources/openviking/openviking_gateway/storage.py:166` `if key := os.environ.get(LEGACY_KEY_ENVIRONMENT):`

</details>

## 5. Reproducibility

Manifest HTTP dependency is unpinned `httpx` (`sources/openviking/examples/hermes-plugin/plugin.yaml:5`); the separate local-engine installer constraints are listed below. A fixed source SHA does not pin an already running remote server. Quick-local engine default is range openviking[local-embed]>=0.5.0,<0.6; wheel resolution may choose an exact version at setup (`sources/openviking/examples/hermes-plugin/quick_local.py:34`; `sources/openviking/examples/hermes-plugin/local_packages.py:138`).

Retained lock/checksum files: `sources/openviking/web-studio/package-lock.json:1`, `sources/openviking/docs/package-lock.json:1`, `sources/openviking/bot/package-lock.json:1`, `sources/openviking/examples/openclaw-plugin/package-lock.json:1`, `sources/openviking/examples/pi-coding-agent-extension/package-lock.json:1`, `sources/openviking/examples/dsh-memory-plugin/package-lock.json:1`, `sources/openviking/sdk/typescript/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/openviking/examples/hermes-plugin/plugin.yaml:2` `version: 2.1.2`
- `sources/openviking/examples/hermes-plugin/plugin.yaml:4` `pip_dependencies:`

- `sources/openviking/examples/hermes-plugin/pyproject.toml:3` `version = "2.1.2"`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:7` `# Installed into the Hermes venv by 'hermes plugins install' / 'enable' and re-applied after`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:9` `# and refuses a plugin whose requirements cannot resolve against them.`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:10` `dependencies = [`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:11` `"httpx>=0.27,<1",`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:12` `"psutil>=5.9,<9",`
- `sources/openviking/examples/hermes-plugin/pyproject.toml:13` `"packaging>=23,<27",`

## 6. Tests

Adapter forget tests exist in `examples/hermes-plugin/tests/test_standalone.py`; scope refusals are tested separately from filesystem deletion. All adapter test-file declaration counts are below. Targeted adapter forget and scope cases are retained. (`sources/openviking/examples/hermes-plugin/tests/test_standalone.py:454`).

Adapter-related declarations: **207**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/openviking/bot/tests/test_openapi_auth.py:1` | 45 | no | `test_delete_rotation_survives_restart_and_session_id_reuse` `sources/openviking/bot/tests/test_openapi_auth.py:359` |
| `sources/openviking/bot/tests/test_session_portable_paths.py:1` | 13 | no | `test_delete_removes_canonical_and_legacy_files` `sources/openviking/bot/tests/test_session_portable_paths.py:181` |
| `sources/openviking/bot/tests/test_studio.py:1` | 17 | no | `test_delete_connection_stops_runtime_and_cleans_owned_data` `sources/openviking/bot/tests/test_studio.py:339` |
| `sources/openviking/bot/tests/test_studio_onboarding.py:1` | 21 | no | `test_manual_recovery_releases_current_job_without_forgetting_app` `sources/openviking/bot/tests/test_studio_onboarding.py:284` |
| `sources/openviking/examples/hermes-plugin/tests/conftest.py:1` | 0 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_env_writes.py:1` | 6 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_extra_headers.py:1` | 5 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_gateway_recall.py:1` | 12 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_local_lifecycle.py:1` | 32 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_local_packages.py:1` | 7 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_native_memory_bridge.py:1` | 6 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_native_memory_mirror.py:1` | 17 | yes | `test_remove_deletes_exact_mapped_uri_and_registry_entry` `sources/openviking/examples/hermes-plugin/tests/test_native_memory_mirror.py:194` |
| `sources/openviking/examples/hermes-plugin/tests/test_quick_local.py:1` | 46 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_quick_local_preflight.py:1` | 10 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_quick_local_routes.py:1` | 11 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_recall_rank.py:1` | 3 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_recall_timeout.py:1` | 7 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_setup_profiles.py:1` | 4 | yes | none |
| `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:1` | 41 | yes | `test_external_provider_accepts_canonical_forget_uris` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:454`; `test_external_provider_rejects_uidless_forget_uris` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:468`; `test_external_provider_rejects_other_user_forget_uri` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:477`; `test_external_provider_forget_fails_closed_without_identity` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:487`; `test_external_provider_rejects_dot_segments_in_forget_uri` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:518`; `test_external_provider_forget_keeps_verified_connection` `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:534` |
| `sources/openviking/examples/openclaw-plugin/tests/ut/architecture-boundaries.test.ts:1` | 95 | no | `keeps memory store and forget handlers out of the composition root` `sources/openviking/examples/openclaw-plugin/tests/ut/architecture-boundaries.test.ts:732` |
| `sources/openviking/examples/openclaw-plugin/tests/ut/plugin-modules.test.ts:1` | 21 | no | `registers memory store and forget through a dedicated plugin module` `sources/openviking/examples/openclaw-plugin/tests/ut/plugin-modules.test.ts:822` |
| `sources/openviking/examples/openclaw-plugin/tests/ut/tools.test.ts:1` | 82 | no | `factory-created forget tool requires either uri or query` `sources/openviking/examples/openclaw-plugin/tests/ut/tools.test.ts:2355` |
| `sources/openviking/integrations/frameworks/langchain/tests/unit/test_langchain_integration.py:1` | 78 | no | `test_chat_message_history_clear_raises_when_delete_fails` `sources/openviking/integrations/frameworks/langchain/tests/unit/test_langchain_integration.py:1597` |
| `sources/openviking/tests/agfs/test_viking_fs_git.py:1` | 38 | no | `test_restore_deletes_marker_and_source_vectors` `sources/openviking/tests/agfs/test_viking_fs_git.py:802`; `test_log_filters_files_directories_and_deletions` `sources/openviking/tests/agfs/test_viking_fs_git.py:215` |
| `sources/openviking/tests/api_test/admin/test_admin_accounts.py:1` | 2 | no | `test_admin_create_delete_account` `sources/openviking/tests/api_test/admin/test_admin_accounts.py:27` |
| `sources/openviking/tests/api_test/scenarios/resources_retrieval/test_delete_sync.py:1` | 1 | no | `test_resource_deletion_index_sync` `sources/openviking/tests/api_test/scenarios/resources_retrieval/test_delete_sync.py:17` |
| `sources/openviking/tests/api_test/scenarios/sessions/test_session_delete_cleanup.py:1` | 1 | no | `test_session_delete_cleanup` `sources/openviking/tests/api_test/scenarios/sessions/test_session_delete_cleanup.py:12` |
| `sources/openviking/tests/api_test/scenarios/slow/test_business_integrity.py:1` | 12 | no | `test_deleted_resource_not_in_ls` `sources/openviking/tests/api_test/scenarios/slow/test_business_integrity.py:129` |
| `sources/openviking/tests/api_test/scenarios/slow/test_concurrency_race.py:1` | 7 | no | `test_delete_session_while_reading_context` `sources/openviking/tests/api_test/scenarios/slow/test_concurrency_race.py:170` |
| `sources/openviking/tests/api_test/scenarios/test_permission_security.py:1` | 7 | no | `test_delete_session_removes_from_list` `sources/openviking/tests/api_test/scenarios/test_permission_security.py:102` |
| `sources/openviking/tests/api_test/sessions/slow/test_session_concurrency.py:1` | 9 | no | `test_delete_session_removes_from_list` `sources/openviking/tests/api_test/sessions/slow/test_session_concurrency.py:68`; `test_delete_nonexistent_session_returns_404` `sources/openviking/tests/api_test/sessions/slow/test_session_concurrency.py:94`; `test_add_message_to_deleted_session_fails` `sources/openviking/tests/api_test/sessions/slow/test_session_concurrency.py:100` |
| `sources/openviking/tests/api_test/sessions/slow/test_session_list_deep.py:1` | 9 | no | `test_list_sessions_after_delete` `sources/openviking/tests/api_test/sessions/slow/test_session_list_deep.py:51` |
| `sources/openviking/tests/api_test/sessions/test_delete_session.py:1` | 1 | no | `test_delete_session` `sources/openviking/tests/api_test/sessions/test_delete_session.py:8` |
| `sources/openviking/tests/cli/test_cli_sessions.py:1` | 7 | no | `test_session_delete` `sources/openviking/tests/cli/test_cli_sessions.py:65` |
| `sources/openviking/tests/client/test_git_versioning_http.py:1` | 7 | no | `test_http_ignore_get_set_delete_roundtrip` `sources/openviking/tests/client/test_git_versioning_http.py:290` |
| `sources/openviking/tests/config/test_runtime_config.py:1` | 56 | no | `test_validate_patch_accepts_mutable_paths_and_none_delete` `sources/openviking/tests/config/test_runtime_config.py:244`; `test_validate_patch_rejects_section_delete_with_locked_descendant` `sources/openviking/tests/config/test_runtime_config.py:251`; `test_nested_delete_preserves_parent_existence` `sources/openviking/tests/config/test_runtime_config.py:286`; `test_delete_account_clears_source_cache_and_notifies_consumers` `sources/openviking/tests/config/test_runtime_config.py:602`; `test_file_source_supports_repeated_update_and_idempotent_delete` `sources/openviking/tests/config/test_runtime_config.py:1229` |
| `sources/openviking/tests/gateway/test_app.py:1` | 39 | no | `test_user_and_account_data_deletion_routes` `sources/openviking/tests/gateway/test_app.py:1369` |
| `sources/openviking/tests/gateway/test_storage_migration.py:1` | 8 | no | `test_user_filtering_and_deletion_preserves_other_users` `sources/openviking/tests/gateway/test_storage_migration.py:279` |
| `sources/openviking/tests/gateway/test_storage_ports.py:1` | 7 | no | `test_cache_coalesces_reads_and_reloads_after_invalidation` `sources/openviking/tests/gateway/test_storage_ports.py:229` |
| `sources/openviking/tests/metrics/collectors/test_state_collectors.py:1` | 12 | no | `test_ragfs_collector_replaces_recovers_and_deletes` `sources/openviking/tests/metrics/collectors/test_state_collectors.py:366` |
| `sources/openviking/tests/metrics/core/test_registry.py:1` | 13 | no | `test_registry_absolute_values_and_deletion` `sources/openviking/tests/metrics/core/test_registry.py:87`; `test_registry_deletion_hides_empty_families_but_keeps_contract` `sources/openviking/tests/metrics/core/test_registry.py:178`; `test_registry_partial_deletion_keeps_active_series` `sources/openviking/tests/metrics/core/test_registry.py:238` |
| `sources/openviking/tests/oc2ov_test/tests/p0/test_memory_crud.py:1` | 3 | no | `test_memory_delete_verify` `sources/openviking/tests/oc2ov_test/tests/p0/test_memory_crud.py:60` |
| `sources/openviking/tests/parse/test_parse_output.py:1` | 33 | no | `test_cleanup_deletes_via_vikingfs` `sources/openviking/tests/parse/test_parse_output.py:214` |
| `sources/openviking/tests/resource/test_watch_manager.py:1` | 46 | no | `test_delete_task` `sources/openviking/tests/resource/test_watch_manager.py:511`; `test_delete_task_not_found` `sources/openviking/tests/resource/test_watch_manager.py:539`; `test_persistence_after_delete` `sources/openviking/tests/resource/test_watch_manager.py:766` |
| `sources/openviking/tests/server/test_account_services.py:1` | 13 | no | `test_file_user_deletion_fence_survives_concurrent_role_update` `sources/openviking/tests/server/test_account_services.py:382`; `test_group_deletion_preserves_users_and_persists_membership_removal` `sources/openviking/tests/server/test_account_services.py:428` |
| `sources/openviking/tests/server/test_admin_api.py:1` | 78 | no | `test_delete_account_files_disables_automatic_pathlock` `sources/openviking/tests/server/test_admin_api.py:318`; `test_delete_account` `sources/openviking/tests/server/test_admin_api.py:2243`; `test_delete_account_retries_when_runtime_config_cleanup_fails` `sources/openviking/tests/server/test_admin_api.py:2423` |
| `sources/openviking/tests/server/test_admin_rebuild_api.py:1` | 69 | no | `test_reindex_rfv_deletes_orphans_synchronously_with_owner_context` `sources/openviking/tests/server/test_admin_rebuild_api.py:826` |
| `sources/openviking/tests/server/test_api_key_manager.py:1` | 38 | no | `test_delete_account` `sources/openviking/tests/server/test_api_key_manager.py:233`; `test_recreated_account_does_not_restore_deleted_users` `sources/openviking/tests/server/test_api_key_manager.py:275`; `test_delete_nonexistent_account_raises` `sources/openviking/tests/server/test_api_key_manager.py:291`; `test_user_deletion_fence_revokes_key_and_rejects_stale_finish` `sources/openviking/tests/server/test_api_key_manager.py:381`; `test_account_deletion_fence_overrides_user_deletion_fence` `sources/openviking/tests/server/test_api_key_manager.py:413` |
| `sources/openviking/tests/server/test_api_privacy_configs.py:1` | 16 | no | `test_privacy_config_delete_is_idempotent_and_removes_history` `sources/openviking/tests/server/test_api_privacy_configs.py:281`; `test_privacy_field_delete_publishes_new_active_snapshot_and_keeps_history` `sources/openviking/tests/server/test_api_privacy_configs.py:478`; `test_privacy_delete_selector_accepts_target_uri` `sources/openviking/tests/server/test_api_privacy_configs.py:538` |
| `sources/openviking/tests/server/test_api_sessions.py:1` | 49 | no | `test_partial_session_remains_visible_and_deletable` `sources/openviking/tests/server/test_api_sessions.py:238`; `test_delete_session` `sources/openviking/tests/server/test_api_sessions.py:1257` |
| `sources/openviking/tests/server/test_api_skills.py:1` | 17 | no | `test_skills_api_list_show_find_and_delete` `sources/openviking/tests/server/test_api_skills.py:74` |
| `sources/openviking/tests/server/test_api_snapshot.py:1` | 24 | no | `test_diff_reports_deleted_path` `sources/openviking/tests/server/test_api_snapshot.py:478`; `test_restore_delete_removes_orphaned_vectors` `sources/openviking/tests/server/test_api_snapshot.py:689` |
| `sources/openviking/tests/server/test_api_watches.py:1` | 14 | no | `test_delete_missing_key_400` `sources/openviking/tests/server/test_api_watches.py:185` |
| `sources/openviking/tests/server/test_api_webdav.py:1` | 11 | no | `test_webdav_move_and_delete_text_file` `sources/openviking/tests/server/test_api_webdav.py:227` |
| `sources/openviking/tests/server/test_identity_auth.py:1` | 10 | no | `test_trusted_unknown_user_cannot_bypass_account_deletion` `sources/openviking/tests/server/test_identity_auth.py:185`; `test_root_context_remains_available_while_default_account_is_deleting` `sources/openviking/tests/server/test_identity_auth.py:199` |
| `sources/openviking/tests/server/test_mcp_endpoint.py:1` | 156 | no | `test_forget_by_uri_deletes_memory` `sources/openviking/tests/server/test_mcp_endpoint.py:2047`; `test_forget_by_uri_deletes_resource` `sources/openviking/tests/server/test_mcp_endpoint.py:2063`; `test_forget_directory_without_recursive_fails` `sources/openviking/tests/server/test_mcp_endpoint.py:2074`; `test_forget_directory_with_recursive_succeeds` `sources/openviking/tests/server/test_mcp_endpoint.py:2085`; `test_forget_rejects_namespace_roots_for_non_root` `sources/openviking/tests/server/test_mcp_endpoint.py:2117` |
| `sources/openviking/tests/server/test_temp_scope_acl.py:1` | 10 | no | `test_delete_temp_removes_repository_tasks_directory` `sources/openviking/tests/server/test_temp_scope_acl.py:212`; `test_non_root_cannot_delete_temp_root_recursively` `sources/openviking/tests/server/test_temp_scope_acl.py:293` |
| `sources/openviking/tests/server/test_upload_token_store.py:1` | 5 | no | `test_purge_expired_drops_stale_tokens` `sources/openviking/tests/server/test_upload_token_store.py:40` |
| `sources/openviking/tests/service/test_fs_service.py:1` | 44 | no | `test_resource_rm_enqueues_parent_delete_refresh_and_waits` `sources/openviking/tests/service/test_fs_service.py:895`; `test_transfer_refresh_message_records_deleted_source_entry` `sources/openviking/tests/service/test_fs_service.py:1418`; `test_resource_rm_does_not_cleanup_memory_if_resource_delete_fails` `sources/openviking/tests/service/test_fs_service.py:1447` |
| `sources/openviking/tests/service/test_resource_memory_link_service.py:1` | 17 | no | `test_on_resource_deleted_bridges_through_fixed_session` `sources/openviking/tests/service/test_resource_memory_link_service.py:414`; `test_before_resource_delete_commits_then_unlinks_stale_refs` `sources/openviking/tests/service/test_resource_memory_link_service.py:608`; `test_unlink_memory_reference_does_not_delete_event_memory` `sources/openviking/tests/service/test_resource_memory_link_service.py:684`; `test_before_resource_delete_cleans_visible_uri_without_resource_refs` `sources/openviking/tests/service/test_resource_memory_link_service.py:720`; `test_before_resource_delete_exact_keeps_child_resource_refs` `sources/openviking/tests/service/test_resource_memory_link_service.py:750` |
| `sources/openviking/tests/service/test_session_service_delete.py:1` | 4 | no | `test_delete_does_not_read_or_parse_session_contents` `sources/openviking/tests/service/test_session_service_delete.py:56`; `test_delete_unregisters_expiry_only_after_removing_session` `sources/openviking/tests/service/test_session_service_delete.py:69`; `test_delete_missing_session_keeps_not_found_semantics` `sources/openviking/tests/service/test_session_service_delete.py:88`; `test_delete_failure_does_not_unregister_expiry` `sources/openviking/tests/service/test_session_service_delete.py:100` |
| `sources/openviking/tests/service/test_task_work_contract.py:1` | 6 | no | `test_deleting_task_releases_retained_work_status` `sources/openviking/tests/service/test_task_work_contract.py:202` |
| `sources/openviking/tests/service/test_watch_recovery.py:1` | 16 | no | `test_task_deactivated_when_resource_deleted` `sources/openviking/tests/service/test_watch_recovery.py:312` |
| `sources/openviking/tests/session/memory/test_consolidation_compile.py:1` | 34 | no | `test_compile_reports_uri_migration_as_add_and_delete` `sources/openviking/tests/session/memory/test_consolidation_compile.py:621` |
| `sources/openviking/tests/session/memory/test_extraction_output_protocol.py:1` | 78 | no | `test_python_delete_replacement_and_links_compile_with_page_id_remapping` `sources/openviking/tests/session/memory/test_extraction_output_protocol.py:1406`; `test_python_update_after_delete_restores_object_and_delete_after_update_discards_update` `sources/openviking/tests/session/memory/test_extraction_output_protocol.py:1454`; `test_python_rejects_delete_for_add_only_schema` `sources/openviking/tests/session/memory/test_extraction_output_protocol.py:1557`; `test_python_rejects_add_only_delete_when_another_schema_enables_deletes` `sources/openviking/tests/session/memory/test_extraction_output_protocol.py:1571` |
| `sources/openviking/tests/session/memory/test_json_stability.py:1` | 39 | no | `test_handles_string_instead_of_list_for_delete` `sources/openviking/tests/session/memory/test_json_stability.py:293` |
| `sources/openviking/tests/session/memory/test_memory_diff.py:1` | 13 | no | `test_build_memory_diff_delete` `sources/openviking/tests/session/memory/test_memory_diff.py:121` |
| `sources/openviking/tests/session/memory/test_memory_react.py:1` | 16 | no | `test_structured_parser_preserves_delete_ids` `sources/openviking/tests/session/memory/test_memory_react.py:572`; `test_add_only_contract_does_not_allow_delete_ids` `sources/openviking/tests/session/memory/test_memory_react.py:645` |
| `sources/openviking/tests/session/memory/test_memory_updater.py:1` | 49 | no | `test_generate_overview_deletes_empty_overview_via_rm` `sources/openviking/tests/session/memory/test_memory_updater.py:268`; `test_generate_overview_skips_deleted_directory` `sources/openviking/tests/session/memory/test_memory_updater.py:310`; `test_apply_operations_skips_deletes_when_replacement_uri_is_unresolved` `sources/openviking/tests/session/memory/test_memory_updater.py:660`; `test_apply_operations_preserves_legacy_delete_suppression_for_expected_skip` `sources/openviking/tests/session/memory/test_memory_updater.py:714`; `test_apply_operations_skips_link_updates_for_deleted_uris` `sources/openviking/tests/session/memory/test_memory_updater.py:806`; `test_apply_operations_skips_case_only_delete_conflicting_with_upsert` `sources/openviking/tests/session/memory/test_memory_updater.py:876`; `test_apply_operations_remaps_deleted_links_to_replacement` `sources/openviking/tests/session/memory/test_memory_updater.py:920` |
| `sources/openviking/tests/session/memory/test_merge_ops.py:1` | 49 | no | `test_apply_dict_delete_patch` `sources/openviking/tests/session/memory/test_merge_ops.py:135`; `test_delete_description_requires_complete_contiguous_lines` `sources/openviking/tests/session/memory/test_merge_ops.py:309`; `test_parse_delete_block_from_json_shape` `sources/openviking/tests/session/memory/test_merge_ops.py:347`; `test_json_schema_exposes_delete_block` `sources/openviking/tests/session/memory/test_merge_ops.py:352`; `test_delete_complete_lines` `sources/openviking/tests/session/memory/test_merge_ops.py:406`; `test_delete_rejects_partial_line` `sources/openviking/tests/session/memory/test_merge_ops.py:411`; `test_duplicate_delete_is_rejected` `sources/openviking/tests/session/memory/test_merge_ops.py:417` |
| `sources/openviking/tests/session/memory/test_schema_models.py:1` | 24 | no | `test_parser_keeps_existing_patch_without_immutable_fields_alongside_delete` `sources/openviking/tests/session/memory/test_schema_models.py:350` |
| `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1` | 39 | no | `test_split_request_by_merge_group_keeps_rename_source_for_add_and_delete` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1012`; `test_split_request_keeps_unresolved_upserts_separate_from_delete_groups` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1086`; `test_template_snapshot_conflicts_cover_adds_and_deletes` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1343`; `test_cross_extraction_merge_preserves_existing_uri_without_explicit_delete` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1821`; `test_force_merge_sends_delete_only_group_through_patch_merge` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1894`; `test_force_merge_does_not_drop_add_only_delete` `sources/openviking/tests/session/memory/test_streaming_memory_updater.py:1944` |
| `sources/openviking/tests/session/train/test_train_components.py:1` | 21 | no | `test_dry_run_policy_updater_simulates_delete_plan_items` `sources/openviking/tests/session/train/test_train_components.py:324`; `test_memory_file_policy_updater_deletes_experience_files` `sources/openviking/tests/session/train/test_train_components.py:634` |
| `sources/openviking/tests/storage/test_collection_schemas.py:1` | 85 | no | `test_embedding_handler_delete_skips_embedder_and_strictly_deletes_ids` `sources/openviking/tests/storage/test_collection_schemas.py:193` |
| `sources/openviking/tests/storage/test_context_update_plan.py:1` | 82 | no | `test_level_conflict_deletes_invalid_record_and_rebuilds_valid_slots` `sources/openviking/tests/storage/test_context_update_plan.py:842`; `test_builder_deletes_duplicate_same_level_records_without_id_conflict` `sources/openviking/tests/storage/test_context_update_plan.py:887`; `test_vectors_only_unchanged_file_with_extra_level_only_deletes_conflict` `sources/openviking/tests/storage/test_context_update_plan.py:1809`; `test_direct_index_actions_delete_stale_record_when_embed_is_skipped` `sources/openviking/tests/storage/test_context_update_plan.py:3033` |
| `sources/openviking/tests/storage/test_embedding_msg.py:1` | 17 | no | `test_embedding_delete_roundtrip` `sources/openviking/tests/storage/test_embedding_msg.py:120` |
| `sources/openviking/tests/storage/test_resource_rfv.py:1` | 24 | no | `test_rfv_missing_directory_sidecar_is_deleted_for_vectors_only_but_repaired_for_semantic` `sources/openviking/tests/storage/test_resource_rfv.py:185` |
| `sources/openviking/tests/storage/test_resource_target.py:1` | 7 | no | `test_delete_path_uses_exact_semantics_for_file` `sources/openviking/tests/storage/test_resource_target.py:77`; `test_delete_path_uses_tree_semantics_for_directory` `sources/openviking/tests/storage/test_resource_target.py:91` |
| `sources/openviking/tests/storage/test_semantic_executor_incremental.py:1` | 25 | no | `test_deleted_last_nested_file_still_refreshes_existing_ancestor` `sources/openviking/tests/storage/test_semantic_executor_incremental.py:643` |
| `sources/openviking/tests/storage/test_session_delete_storage.py:1` | 10 | no | `test_session_rm_releases_its_tree_lease_when_backend_delete_fails` `sources/openviking/tests/storage/test_session_delete_storage.py:132` |
| `sources/openviking/tests/storage/test_snapshot_deleted_file_binding.py:1` | 3 | no | `test_commit_deleted_file_stays_absent` `sources/openviking/tests/storage/test_snapshot_deleted_file_binding.py:42`; `test_commit_deleted_directory_stays_absent` `sources/openviking/tests/storage/test_snapshot_deleted_file_binding.py:72` |
| `sources/openviking/tests/storage/test_vector_migration.py:1` | 5 | no | `test_delete_vector_records_deletes_records_in_scope_only` `sources/openviking/tests/storage/test_vector_migration.py:146` |
| `sources/openviking/tests/storage/test_vector_transfer.py:1` | 53 | no | `test_strict_delete_removes_existing_subset_when_attempted_ids_include_missing` `sources/openviking/tests/storage/test_vector_transfer.py:422`; `test_copy_uri_mapping_reports_rollback_when_delete_returns_zero` `sources/openviking/tests/storage/test_vector_transfer.py:1674`; `test_update_uri_mapping_deletes_source_only_after_targets_exist` `sources/openviking/tests/storage/test_vector_transfer.py:1694`; `test_update_uri_mapping_restores_source_and_removes_target_after_partial_delete` `sources/openviking/tests/storage/test_vector_transfer.py:1744` |
| `sources/openviking/tests/storage/test_viking_fs_cp.py:1` | 29 | no | `test_mv_keeps_target_when_source_delete_fails` `sources/openviking/tests/storage/test_viking_fs_cp.py:615`; `test_mv_directory_keeps_target_after_partial_source_delete` `sources/openviking/tests/storage/test_viking_fs_cp.py:711`; `test_mv_does_not_copy_back_after_source_delete_failure` `sources/openviking/tests/storage/test_viking_fs_cp.py:807` |
| `sources/openviking/tests/storage/test_viking_fs_record_ids.py:1` | 1 | no | `test_missing_record_id_explains_index_or_deletion_state` `sources/openviking/tests/storage/test_viking_fs_record_ids.py:21` |
| `sources/openviking/tests/storage/test_volcengine_clients.py:1` | 37 | no | `test_volcengine_delete_propagates_request_failures` `sources/openviking/tests/storage/test_volcengine_clients.py:331` |
| `sources/openviking/tests/test_edge_cases.py:1` | 24 | no | `test_search_after_delete` `sources/openviking/tests/test_edge_cases.py:163`; `test_rapid_create_delete_cycles` `sources/openviking/tests/test_edge_cases.py:301` |
| `sources/openviking/tests/test_task_tracker.py:1` | 63 | no | `test_delete_expired_retries_after_persistent_delete_fails` `sources/openviking/tests/test_task_tracker.py:570` |
| `sources/openviking/tests/unit/session/memory/test_extract_loop_match_text.py:1` | 31 | no | `test_delete_with_ambiguous_new_replacement_page_id_is_skipped` `sources/openviking/tests/unit/session/memory/test_extract_loop_match_text.py:432` |
| `sources/openviking/tests/utils/test_resource_processor_processing_mode.py:1` | 24 | no | `test_add_resources_rnfv_uses_subtree_inventory_to_delete_orphans` `sources/openviking/tests/utils/test_resource_processor_processing_mode.py:324`; `test_vectors_only_deletes_sync_removed_detail_vectors` `sources/openviking/tests/utils/test_resource_processor_processing_mode.py:1088`; `test_delete_removed_resource_vectors_deletes_detail_records` `sources/openviking/tests/utils/test_resource_processor_processing_mode.py:1141` |
| `sources/openviking/tests/vectordb/test_collection_large_scale.py:1` | 10 | no | `test_05_massive_deletes` `sources/openviking/tests/vectordb/test_collection_large_scale.py:345` |
| `sources/openviking/tests/vectordb/test_cuvs_collection.py:1` | 20 | no | `test_auto_cuvs_bulk_ingest_delete_all_replacement_stays_suspended` `sources/openviking/tests/vectordb/test_cuvs_collection.py:1313` |
| `sources/openviking/tests/vectordb/test_cuvs_index.py:1` | 71 | no | `test_cuvs_dense_search_handles_filter_upsert_delete_and_lazy_rebuild` `sources/openviking/tests/vectordb/test_cuvs_index.py:852`; `test_filter_cache_reuses_prepared_mask_and_invalidates_on_mutation` `sources/openviking/tests/vectordb/test_cuvs_index.py:2585` |
| `sources/openviking/tests/vectordb/test_filter_ops.py:1` | 28 | no | `test_delete_impact` `sources/openviking/tests/vectordb/test_filter_ops.py:556` |
| `sources/openviking/tests/vectordb/test_opengauss_adapter_complete.py:1` | 67 | no | `test_filter_delete_executes_direct_sql_without_implicit_limit` `sources/openviking/tests/vectordb/test_opengauss_adapter_complete.py:1329`; `test_delete_index_meta_tolerates_only_missing_catalog_table` `sources/openviking/tests/vectordb/test_opengauss_adapter_complete.py:1596` |
| `sources/openviking/tests/vectordb/test_opengauss_review_regressions.py:1` | 8 | no | `test_delete_ids_matches_upstream_contract` `sources/openviking/tests/vectordb/test_opengauss_review_regressions.py:124`; `test_filter_delete_has_no_implicit_row_limit` `sources/openviking/tests/vectordb/test_opengauss_review_regressions.py:140` |
| `sources/openviking/tests/vectordb/test_openviking_vectordb.py:1` | 3 | no | `test_filters_update_delete_recall` `sources/openviking/tests/vectordb/test_openviking_vectordb.py:251`; `test_offset_collision_after_delete` `sources/openviking/tests/vectordb/test_openviking_vectordb.py:461` |
| `sources/openviking/web-studio/src/components/sessions/delete-conversation.test.tsx:1` | 2 | no | `requires confirmation and does not delete on cancel` `sources/openviking/web-studio/src/components/sessions/delete-conversation.test.tsx:29`; `does not dismiss the conversation when deletion fails` `sources/openviking/web-studio/src/components/sessions/delete-conversation.test.tsx:49` |
| `sources/openviking/web-studio/src/components/studio-setup.test.tsx:1` | 16 | no | `enters only after verification, invalidates verification when fields change, and persists on explicit entry` `sources/openviking/web-studio/src/components/studio-setup.test.tsx:395` |
| `sources/openviking/web-studio/src/routes/filesystem/-components/agent-panel-submission.test.tsx:1` | 7 | no | `deletes the active history session only after confirmation and returns to a draft` `sources/openviking/web-studio/src/routes/filesystem/-components/agent-panel-submission.test.tsx:158` |
| `sources/openviking/web-studio/src/routes/filesystem/route.test.tsx:1` | 3 | no | `clears a deleted selected file from the preview and URL search` `sources/openviking/web-studio/src/routes/filesystem/route.test.tsx:149` |
| `sources/openviking/web-studio/src/routes/gateway/-components/keys-page.test.tsx:1` | 7 | no | `hides the user column and user-data deletion for ordinary users` `sources/openviking/web-studio/src/routes/gateway/-components/keys-page.test.tsx:282` |
| `sources/openviking/web-studio/src/routes/gateway/-components/profiles-pages.test.tsx:1` | 26 | no | `blocks deleting a profile that keys use and deletes an unused one` `sources/openviking/web-studio/src/routes/gateway/-components/profiles-pages.test.tsx:300` |
| `sources/openviking/web-studio/src/routes/gateway/-components/upstreams-pages.test.tsx:1` | 33 | no | `deletes an unused upstream after confirmation` `sources/openviking/web-studio/src/routes/gateway/-components/upstreams-pages.test.tsx:497`; `explains deleting an upstream that keys use` `sources/openviking/web-studio/src/routes/gateway/-components/upstreams-pages.test.tsx:512` |
| `sources/openviking/web-studio/src/routes/gateway/-lib/api.test.ts:1` | 11 | no | `addresses resync, key issuance and user data deletion` `sources/openviking/web-studio/src/routes/gateway/-lib/api.test.ts:107` |
| `sources/openviking/web-studio/src/routes/gateway/-lib/use-gateway.test.tsx:1` | 8 | no | `loads resources with the admin connection and invalidates by resource` `sources/openviking/web-studio/src/routes/gateway/-lib/use-gateway.test.tsx:146` |
| `sources/openviking/web-studio/src/routes/settings/-components/server-configuration.test.tsx:1` | 36 | no | `stages additions and confirmed deletions, cancels edits and undoes the draft` `sources/openviking/web-studio/src/routes/settings/-components/server-configuration.test.tsx:301` |
| `sources/openviking/web-studio/src/routes/users/-components/delete-account-button.test.tsx:1` | 2 | no | `requires the exact account name before deleting` `sources/openviking/web-studio/src/routes/users/-components/delete-account-button.test.tsx:82`; `clears the deleted identity and opens settings when no account remains` `sources/openviking/web-studio/src/routes/users/-components/delete-account-button.test.tsx:114` |
| `sources/openviking/web-studio/src/routes/users/-components/user-groups.test.tsx:1` | 4 | no | `blocks nonempty deletion and requires confirmation for empty groups` `sources/openviking/web-studio/src/routes/users/-components/user-groups.test.tsx:85`; `keeps a failed deletion open and refreshes potentially stale member counts` `sources/openviking/web-studio/src/routes/users/-components/user-groups.test.tsx:160` |
| `sources/openviking/web-studio/src/routes/users/-components/user-memory-policy-cell.test.tsx:1` | 3 | no | `refreshes mounted policies when the account prefix is invalidated` `sources/openviking/web-studio/src/routes/users/-components/user-memory-policy-cell.test.tsx:121` |
| `sources/openviking/web-studio/src/routes/watches/-lib/api.test.ts:1` | 6 | no | `deletes a watch by encoded task ID` `sources/openviking/web-studio/src/routes/watches/-lib/api.test.ts:102` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `forget_tool.scope`: `"viking_forget removes URI; session archive/memory derivation cleanup must be checked independently."` → Only concrete own-user memory .md URIs; summaries and broad paths refused; parent semantic refresh may be deferred. Evidence: `sources/openviking/examples/hermes-plugin/__init__.py:3346`, `sources/openviking/openviking/service/fs_service.py:545`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: `viking_forget` validates one concrete own-user `.md` URI, rejects generated summaries and broad paths, then DELETE `/api/v1/fs` with `recursive=False` (`sources/openviking/examples/hermes-plugin/__init__.py:3346`; `sources/openviking/examples/hermes-plugin/__init__.py:3349`; `sources/openviking/examples/hermes-plugin/__init__.py:645`). Engine `fs_service.rm` delegates to `_rm`, which removes filesystem data, synchronizes watches, schedules semantic refresh, cleans resource-memory links, and refreshes memory overviews (`sources/openviking/openviking/service/fs_service.py:498`). `partial`: refresh can be deferred and independent session archives/source text are not all erased by deleting one memory file (`sources/openviking/openviking/service/fs_service.py:509`; `sources/openviking/openviking/service/fs_service.py:545`). Other engine removal/session/admin routes are in the inventory. Evidence: `sources/openviking/examples/hermes-plugin/__init__.py:3346`, `sources/openviking/examples/hermes-plugin/__init__.py:3349`, `sources/openviking/examples/hermes-plugin/__init__.py:645`, `sources/openviking/openviking/service/fs_service.py:498`, `sources/openviking/openviking/service/fs_service.py:509`, `sources/openviking/openviking/service/fs_service.py:545`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → `system_prompt_block` supplies usage instructions; `prefetch` joins startup context and query recall as `## OpenViking Context` (`sources/openviking/examples/hermes-plugin/__init__.py:1991`; `sources/openviking/examples/hermes-plugin/__init__.py:2008`). These are separate contributions, so a per-search cap is not a total context cap. Tool output is JSON through the dispatch handler (`sources/openviking/examples/hermes-plugin/__init__.py:3171`). Exact assembly helpers and limit expressions below. Evidence: `sources/openviking/examples/hermes-plugin/__init__.py:1991`, `sources/openviking/examples/hermes-plugin/__init__.py:2008`, `sources/openviking/examples/hermes-plugin/__init__.py:3171`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → Direct access `yes`: the same API is callable without the adapter; local backing paths are also accessible to their OS owner. API keys/model keys are configuration-specific, not protected by URI validation in the adapter (`sources/openviking/examples/hermes-plugin/__init__.py:3346`). Profile `.env`, client/server config and local runtime paths/modes are in the source ledger. Accountless local mode `yes` with locally configured models; no claim that model downloads or defaults are offline. Telemetry: no universal default established across optional server/model dependencies. Evidence: `sources/openviking/examples/hermes-plugin/__init__.py:3346`.
- `repro.decls`: `["Cargo.toml:7", "pyproject.toml:3", "pyproject.toml:4", "pyproject.toml:5", "pyproject.toml:6", "pyproject.toml:13", "pyproject.toml:15", "pyproject.toml:31", "pyproject.toml:32", "pyproject.toml:33", "pyproject.toml:34", "pyproject.toml:35", "pyproject.toml:36", "pyproject.toml:37", "pyproject.toml:38", "pyproject.toml:39", "pyproject.toml:40", "pyproject.toml:41", "pyproject.toml:42", "pyproject.toml:43", "pyproject.toml:44", "pyproject.toml:45", "pyproject.toml:46", "pyproject.toml:47", "pyproject.toml:48", "pyproject.toml:49", "pyproject.toml:50", "pyproject.toml:51", "pyproject.toml:52", "pyproject.toml:53", "pyproject.toml:54", "pyproject.toml:55", "pyproject.toml:56", "pyproject.toml:57", "pyproject.toml:58", "pyproject.toml:59", "pyproject.toml:60", "pyproject.toml:61", "pyproject.toml:62", "pyproject.toml:63", "pyproject.toml:64", "pyproject.toml:65", "pyproject.toml:66", "pyproject.toml:68", "pyproject.toml:69", "pyproject.toml:70", "pyproject.toml:71", "pyproject.toml:72", "pyproject.toml:73", "pyproject.toml:74", "pyproject.toml:75", "pyproject.toml:76", "pyproject.toml:77", "pyproject.toml:78", "pyproject.toml:80", "pyproject.toml:81", "pyproject.toml:82", "pyproject.toml:83", "pyproject.toml:84", "pyproject.toml:85", "pyproject.toml:86", "pyproject.toml:87", "pyproject.toml:88", "pyproject.toml:89", "pyproject.toml:90"]` → Manifest HTTP dependency is unpinned `httpx` (`sources/openviking/examples/hermes-plugin/plugin.yaml:5`); the separate local-engine installer constraints are listed below. A fixed source SHA does not pin an already running remote server. Quick-local engine default is range openviking[local-embed]>=0.5.0,<0.6; wheel resolution may choose an exact version at setup (`sources/openviking/examples/hermes-plugin/quick_local.py:34`; `sources/openviking/examples/hermes-plugin/local_packages.py:138`). Evidence: `sources/openviking/examples/hermes-plugin/plugin.yaml:5`, `sources/openviking/examples/hermes-plugin/quick_local.py:34`, `sources/openviking/examples/hermes-plugin/local_packages.py:138`.
- `tests.count`: `12656` → 207 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter forget tests exist in `examples/hermes-plugin/tests/test_standalone.py`; scope refusals are tested separately from filesystem deletion. All adapter test-file declaration counts are below. Targeted adapter forget and scope cases are retained. (`sources/openviking/examples/hermes-plugin/tests/test_standalone.py:454`). Evidence: `sources/openviking/examples/hermes-plugin/tests/test_standalone.py:454`.

MATRIX counts these 6 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
