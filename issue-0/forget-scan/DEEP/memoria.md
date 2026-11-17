# memoria: B3 static correctness review

Snapshot `3a5c5065b2820d56dd3d8ac8bb259a9424b1476e` (MANIFEST.json row `memoria`). No plugin execution or passing-test claims.

## 1. Architecture

`provider`: HTTP integration to Memoria service with durable capture and a read/recall cache. Snapshot also includes Rust engine and Python SDK; adapter directly issues HTTP rather than using that SDK (`sources/memoria/plugins/hermes/__init__.py:561`; `sources/memoria/plugins/hermes/__init__.py:609`).

Hook/body inventory (no-op/fallback declarations are not active behavior):

| callable | declaration | first executable body line |
|---|---|---|
| `register` | `sources/memoria/plugins/hermes/__init__.py:728` | `sources/memoria/plugins/hermes/__init__.py:729` |
| `is_available` | `sources/memoria/plugins/hermes/__init__.py:225` | `sources/memoria/plugins/hermes/__init__.py:226` |
| `initialize` | `sources/memoria/plugins/hermes/__init__.py:240` | `sources/memoria/plugins/hermes/__init__.py:241` |
| `system_prompt_block` | `sources/memoria/plugins/hermes/__init__.py:288` | `sources/memoria/plugins/hermes/__init__.py:289` |
| `prefetch` | `sources/memoria/plugins/hermes/__init__.py:323` | `sources/memoria/plugins/hermes/__init__.py:324` |
| `queue_prefetch` | `sources/memoria/plugins/hermes/__init__.py:363` | `sources/memoria/plugins/hermes/__init__.py:364` |
| `on_session_switch` | `sources/memoria/plugins/hermes/__init__.py:404` | `sources/memoria/plugins/hermes/__init__.py:405` |
| `sync_turn` | `sources/memoria/plugins/hermes/__init__.py:409` | `sources/memoria/plugins/hermes/__init__.py:412` |
| `get_tool_schemas` | `sources/memoria/plugins/hermes/__init__.py:542` | `sources/memoria/plugins/hermes/__init__.py:544` |
| `handle_tool_call` | `sources/memoria/plugins/hermes/__init__.py:546` | `sources/memoria/plugins/hermes/__init__.py:547` |
| `shutdown` | `sources/memoria/plugins/hermes/__init__.py:660` | `sources/memoria/plugins/hermes/__init__.py:661` |
| `get_config_schema` | `sources/memoria/plugins/hermes/__init__.py:672` | `sources/memoria/plugins/hermes/__init__.py:673` |
| `save_config` | `sources/memoria/plugins/hermes/__init__.py:690` | `sources/memoria/plugins/hermes/__init__.py:691` |

## 2. Forget paths and derived copies

`yes`: memoria_forget validates the ID, GETs it in the configured branch, checks subject_id, then DELETE `/v1/memories/{id}` (`sources/memoria/plugins/hermes/__init__.py:598`; `sources/memoria/plugins/hermes/__init__.py:609`). memoria_update uses PUT `/v1/memories/{id}/correct`, a separate correction operation (`sources/memoria/plugins/hermes/__init__.py:603`). Writes invalidate adapter cache (`sources/memoria/plugins/hermes/__init__.py:625`). Rust delete calls purge_on_branch (`sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:613`). Purge supports IDs, topic and session selectors (`sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:628`). Derived reach `partial`: branch-local purge and graph/SQL cleanup are not a promise to remove independent snapshots, branches, exported copies or host history.

Host state.db is independent (`hermes-agent/hermes_state.py:181`). No reviewed adapter forget body performs a general purge of that database or built-in MEMORY.md/USER.md. Prompt/tool-result copies and independent source transcripts remain separate; source operation bodies below establish the scope of each plugin mutation.

<details>
<summary>Deletion, invalidation, archive and reset body ledger</summary>

Public routes, SDK/CLI dispatch and internal storage operations are distinguished by their source. Cache invalidation and management-object deletion are not additional fact-forget tools. A stub is not working deletion. Call-through means the downstream body still determines erasure. HTTP prefixes/selectors must be read with route registration.

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/auth.rs:346`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:345` `.unwrap_or(false);`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:346` `if req.method() == axum::http::Method::DELETE`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:347` `&& path.starts_with("/v1/memories/")`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:348` `&& !path.ends_with("/correct")`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:349` `&& explicit_query_branch`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:350` `{`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:357` `| "/v1/memories/correct"`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:358` `| "/v1/memories/purge"`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:359` `| "/v1/observe"`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:360` `| "/v1/observe/deduplicated"`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:361` `) || (path.starts_with("/v1/memories/")`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:362` `&& path.ends_with("/correct"));`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1327` `if fresh {`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1328` `state.api_key_cache.invalidate(&key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1329` `} else if let Some(principal) = state.api_key_cache.get(&key_hash) {`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1330` `// Still enqueue last_used_at update (batched, no DB pressure)`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1331` `state.last_used_batcher.mark_used(key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1332` `return Some(principal);`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1446` `assert_eq!(`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1447` `required_scope_for_request(&axum::http::Method::DELETE, "/v1/snapshots/one"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1448` `Some(SCOPE_MEMORY_WRITE)`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1449` `);`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1450` `assert_eq!(`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1451` `required_scope_for_request(&axum::http::Method::GET, "/auth/whoami"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1468` `(Method::POST, "/v1/groups/group/members/another-user"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1469` `(Method::DELETE, "/v1/groups/group"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1470` `(Method::DELETE, "/v1/groups/group/members/another-user"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1471` `(Method::GET, "/unclassified-sensitive-route"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1472` `(Method::POST, "/admin/users"),`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1473` `] {`
- `sources/memoria/memoria/crates/memoria-api/src/auth.rs:1474` `assert_eq!(`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/lib.rs:19`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:18` `response::IntoResponse,`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:19` `routing::{delete, get, post, put},`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:20` `Json, Router,`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:21` `};`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:22` `use tower_http::catch_panic::CatchPanicLayer;`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:23` ``
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:130` `| "/v1/memories/correct"`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:131` `| "/v1/memories/purge"`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:132` `| "/v1/observe"`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:133` `| "/v1/observe/deduplicated"`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:134` `| "/v1/pipeline/run" => Some(DirtyMask::MEMORY),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:135` `"/v1/governance"`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:139` `| "/v1/extract-entities/link" => Some(DirtyMask::MEMORY | DirtyMask::GRAPH),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:140` `"/v1/snapshots" | "/v1/snapshots/delete" => Some(DirtyMask::SNAPSHOT),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:141` `"/v1/branches" => Some(DirtyMask::BRANCH),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:142` `_ => {`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:143` `// Dynamic path patterns`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:144` `if path.starts_with("/v1/memories/") && path.ends_with("/feedback") {`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:162` ``
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:163` `if method == "DELETE" {`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:164` `if path.starts_with("/v1/memories/") {`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:165` `return Some(DirtyMask::MEMORY);`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:166` `}`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:167` `if path.starts_with("/v1/snapshots/") {`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:202` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:203` `.route("/v1/memories/purge", post(routes::memory::purge_memories))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:204` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:205` `"/v1/memories/:id/correct",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:206` `put(routes::memory::correct_memory),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:207` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:208` `.route("/v1/memories/:id", delete(routes::memory::delete_memory))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:209` `.route("/v1/observe", post(routes::memory::observe_turn))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:210` `.route("/v1/observe/deduplicated", post(routes::memory::observe_turn_deduplicated))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:211` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:212` `"/v1/sessions/:session_id/summary",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:297` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:298` `"/v1/snapshots/delete",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:299` `post(routes::snapshots::delete_snapshot_bulk),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:300` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:301` `.route("/v1/snapshots/:name", get(routes::snapshots::get_snapshot))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:302` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:303` `"/v1/snapshots/:name",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:304` `delete(routes::snapshots::delete_snapshot),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:305` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:306` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:307` `"/v1/snapshots/:name/rollback",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:308` `post(routes::snapshots::rollback),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:338` `"/v1/branches/:name",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:339` `delete(routes::snapshots::delete_branch),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:340` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:341` `// Sessions`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:342` `.route("/v1/tasks/:task_id", get(routes::sessions::get_task_status))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:343` `// API key management`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:348` `.route("/auth/keys/:id/rotate", put(routes::auth::rotate_key))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:349` `.route("/auth/keys/:id", delete(routes::auth::revoke_key))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:350` `// Group management (user-facing)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:351` `.route("/v1/groups", post(routes::groups::create_group))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:352` `.route("/v1/groups", get(routes::groups::list_my_groups))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:353` `.route("/v1/groups/:group_id", get(routes::groups::get_group))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:359` `"/v1/groups/:group_id/members/:user_id",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:360` `delete(routes::groups::remove_member),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:361` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:362` `.route("/v1/groups/:group_id", delete(routes::groups::delete_group))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:363` `// Merge write routes and merge-guarded route`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:364` `.merge(write_routes)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:365` `.merge(merge_route)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:366` `// Admin`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:381` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:382` `.route("/admin/users/:user_id", delete(routes::admin::delete_user))`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:383` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:384` `"/admin/users/:user_id/keys",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:385` `get(routes::admin::list_user_keys),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:386` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:388` `"/admin/users/:user_id/keys",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:389` `delete(routes::admin::revoke_all_user_keys),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:390` `)`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:391` `.route(`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:392` `"/admin/users/:user_id/params",`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:393` `post(routes::admin::set_user_params),`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:517` `assert!(should_mark_metrics_dirty("POST", "/v1/snapshots/demo/rollback", 200).is_some());`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:518` `assert!(should_mark_metrics_dirty("DELETE", "/v1/branches/demo", 204).is_some());`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:519` `assert!(should_mark_metrics_dirty("POST", "/v1/sessions/abc/summary", 200).is_some());`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:520` `}`
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:521` ``
- `sources/memoria/memoria/crates/memoria-api/src/lib.rs:522` `#[test]`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1171`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1170` `.join(",");`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1171` `let delete_sql = format!(`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1172` `"DELETE FROM mem_metrics_user_rollups WHERE family = ? AND user_id IN ({user_placeholders})"`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1173` `);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1174` `let mut del_query = sqlx::query(&delete_sql).bind(family);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1175` `for uid in &refreshing_user_ids {`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1176` `del_query = del_query.bind(*uid);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1177` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1178` `del_query.execute(&self.shared_pool).await.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1374` ``
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1375` `self.invalidate_metrics_cache().await;`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1376` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1377` ``
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1378` `self.stats`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1379` `.last_duration_ms`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1408` ``
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1409` `async fn invalidate_metrics_cache(&self) {`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1410` `let mut cache = self.metrics_cache.write().await;`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1411` `*cache = None;`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1412` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1413` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1865` `#[test]`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1866` `fn cross_family_deletion_does_not_leak() {`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1867` `// Scenario from the review: user A refreshes MEMORY, user B refreshes`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1868` `// FEEDBACK only.  The memory_type DELETE must NOT include user B.`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1869` `let batch: Vec<(ClaimedUser, Vec<RollupEntry>)> = vec![`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1870` `(`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1887` `!users.contains(&"B"),`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1888` `"user B must NOT be in the delete set for {fam_name}"`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1889` `);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1890` `assert!(`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1891` `users.contains(&"A"),`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1892` `"user A must be in the delete set for {fam_name}"`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1893` `);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1894` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1895` ``
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1896` `// For the FEEDBACK family, only B should be in the delete set`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1899` `!fb_users.contains(&"A"),`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1900` `"user A must NOT be in the delete set for feedback_signal"`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1901` `);`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1902` `assert!(fb_users.contains(&"B"));`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1903` `}`
- `sources/memoria/memoria/crates/memoria-api/src/metrics_summary.rs:1904` ``

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/models.rs:367`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/models.rs:366` `#[derive(Deserialize)]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:367` `pub struct PurgeRequest {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:368` `pub memory_ids: Option<Vec<String>>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:369` `pub topic: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:370` `pub session_id: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:371` `pub memory_types: Option<Vec<String>>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:375` ``
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:376` `pub enum PurgeSelector {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:377` `MemoryIds(Vec<String>),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:378` `Topic(String),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:379` `Session {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:380` `session_id: String,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:385` ``
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:386` `impl PurgeRequest {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:387` `pub fn selector(&self) -> Result<PurgeSelector, String> {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:388` `let memory_ids = self.memory_ids.as_ref().filter(|ids| !ids.is_empty());`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:389` `let memory_types = self.memory_types.as_ref().filter(|types| !types.is_empty());`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:390` `let topic = self`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:391` `.topic`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:411` `if let Some(ids) = memory_ids {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:412` `return Ok(PurgeSelector::MemoryIds(ids.clone()));`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:413` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:414` `if let Some(topic) = topic {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:415` `return Ok(PurgeSelector::Topic(topic.to_string()));`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:416` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:417` `if let Some(session_id) = session_id {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:418` `let memory_types = memory_types`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:419` `.map(|types| {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:426` `.filter(|types| !types.is_empty());`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:427` `return Ok(PurgeSelector::Session {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:428` `session_id: session_id.to_string(),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:429` `memory_types,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:430` `});`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:431` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:432` `Ok(PurgeSelector::None)`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:433` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:434` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:435` ``
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:436` `#[derive(Serialize)]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:488` `#[derive(Serialize)]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:489` `pub struct PurgeResponse {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:490` `pub purged: usize,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:491` `#[serde(skip_serializing_if = "Option::is_none")]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:492` `pub snapshot_name: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:493` `#[serde(skip_serializing_if = "Option::is_none")]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:494` `pub warning: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:551` `#[derive(Deserialize)]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:552` `pub struct DeleteSnapshotsRequest {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:553` `pub names: Option<Vec<String>>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:554` `pub prefix: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:555` `pub older_than: Option<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:556` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:666` `use super::{`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:667` `parse_fulltext_memory_types_opt, parse_memory_types_opt, PurgeRequest, PurgeSelector,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:668` `};`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:669` `use memoria_core::MemoryType;`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:670` ``
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:671` `#[test]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:694` `#[test]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:695` `fn purge_selector_ignores_empty_arrays() {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:696` `let request = PurgeRequest {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:697` `memory_ids: Some(vec![]),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:698` `topic: None,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:699` `session_id: Some("sess-1".to_string()),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:700` `memory_types: Some(vec![]),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:705` `match request.selector().unwrap() {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:706` `PurgeSelector::Session {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:707` `session_id,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:708` `memory_types,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:709` `} => {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:710` `assert_eq!(session_id, "sess-1");`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:717` `#[test]`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:718` `fn purge_selector_empty_memory_types_do_not_require_session() {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:719` `let request = PurgeRequest {`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:720` `memory_ids: None,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:721` `topic: None,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:722` `session_id: None,`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:723` `memory_types: Some(vec![]),`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:727` ``
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:728` `assert!(matches!(request.selector().unwrap(), PurgeSelector::None));`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:729` `}`
- `sources/memoria/memoria/crates/memoria-api/src/models.rs:730` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:201`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:200` `/// DELETE /admin/users/:user_id`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:201` `pub async fn delete_user(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:202` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:203` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:204` `Path(user_id): Path<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:205` `) -> Result<Json<serde_json::Value>, (StatusCode, String)> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:222` `error = %e,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:223` `"failed to mark metrics summary dirty after admin delete_user"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:224` `);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:225` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:226` `Ok(Json(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:227` `serde_json::json!({"status": "ok", "user_id": user_id}),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:266` `let cleaned_tool_results = sql.cleanup_tool_results(72).await.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:267` `let archived_working = sql.archive_stale_working(24).await.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:268` `let compressed = sql`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:269` `.compress_redundant(&user_id, 0.95, 30, 10_000)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:270` `.await`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:271` `.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:284` `"cleaned_tool_results": cleaned_tool_results,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:285` `"archived_working": archived_working,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:286` `"compressed_redundant": compressed,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:287` `"cleaned_incrementals": cleaned_incrementals,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:288` `"pollution_detected": pollution_detected,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:289` `"orphan_graph_cleaned": orphan_graph_cleaned,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:729` `THEN 1 ELSE 0 END) AS SIGNED) AS retrieves, \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:730` `CAST(SUM(CASE WHEN (method = 'DELETE' AND path LIKE '/v1/memories/%') \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:731` `OR path = '/mcp/memory_purge' \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:732` `THEN 1 ELSE 0 END) AS SIGNED) AS deletes, \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:733` `CAST(COALESCE(AVG(CASE WHEN path IN ('/v1/memories/search','/v1/memories/retrieve', \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:734` `'/mcp/memory_retrieve','/mcp/memory_search') \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:735` `THEN latency_ms END), 0) AS DOUBLE) AS avg_retrieval_ms \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:736` `FROM {call_log} \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:746` `let at_retrieves: i64 = at_row.try_get("retrieves").unwrap_or(0);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:747` `let at_deletes: i64 = at_row.try_get("deletes").unwrap_or(0);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:748` `let at_avg_ret: f64 = at_row.try_get("avg_retrieval_ms").unwrap_or(0.0);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:749` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:750` `// ── Per-day series within the requested window ─────────────────────────────`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:751` `// Used by the Usage panel's API Call Tracking chart.`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:759` `THEN 1 ELSE 0 END) AS SIGNED) AS writes, \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:760` `CAST(SUM(CASE WHEN (method = 'DELETE' AND path LIKE '/v1/memories/%') \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:761` `OR path = '/mcp/memory_purge' \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:762` `THEN 1 ELSE 0 END) AS SIGNED) AS deletes, \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:763` `CAST(SUM(CASE WHEN path IN ('/v1/memories/search','/v1/memories/retrieve', \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:764` `'/mcp/memory_retrieve','/mcp/memory_search') \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:765` `THEN 1 ELSE 0 END) AS SIGNED) AS retrieves, \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:766` `CAST(COUNT(*) AS SIGNED) AS total \`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:785` `"writes":    r.try_get::<i64,_>("writes").unwrap_or(0),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:786` `"deletes":   r.try_get::<i64,_>("deletes").unwrap_or(0),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:787` `"retrieves": r.try_get::<i64,_>("retrieves").unwrap_or(0),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:788` `"total":     r.try_get::<i64,_>("total").unwrap_or(0),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:789` `})`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:790` `})`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:808` `"total_retrieves": at_retrieves,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:809` `"total_deletes":   at_deletes,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:810` `"avg_retrieval_ms": at_avg_ret as i64,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:811` `},`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:812` `"series": series,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/admin.rs:813` `})))`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:391`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:390` `if let Ok(key_hash) = old.try_get::<String, _>("key_hash") {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:391` `state.api_key_cache.invalidate(&key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:392` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:393` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:394` `// Deactivate old`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:395` `sqlx::query("UPDATE mem_api_keys SET is_active = 0 WHERE key_id = ?")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:477` `if let Ok(key_hash) = row.try_get::<String, _>("key_hash") {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:478` `state.api_key_cache.invalidate(&key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:479` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:480` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:481` `sqlx::query("UPDATE mem_api_keys SET is_active = 0 WHERE key_id = ?")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:482` `.bind(&key_id)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:258`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:257` `async fn cleanup_failed_group_creation(pool: &MySqlPool, group_id: &str, db_name: &str) {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:258` `let _ = sqlx::query("DELETE FROM mem_group_members WHERE group_id = ?")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:259` `.bind(group_id)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:260` `.execute(pool)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:261` `.await;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:262` `let _ = sqlx::query("DELETE FROM mem_groups WHERE group_id = ?")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:263` `.bind(group_id)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:264` `.execute(pool)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:265` `.await;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:266` `let safe_db = db_name.replace(''', "''");`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:591` `if let Ok(key_hash) = row.try_get::<String, _>("key_hash") {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:592` `state.api_key_cache.invalidate(&key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:593` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:594` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:595` `sqlx::query(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:596` `"UPDATE mem_api_keys SET is_active = 0 WHERE group_id = ? AND user_id = ? AND is_active = 1",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:611` `/// DELETE /v1/groups/:group_id , soft-delete a group (owner only)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:612` `pub async fn delete_group(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:613` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:614` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:615` `Path(group_id): Path<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:616` `) -> Result<Json<serde_json::Value>, (StatusCode, String)> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:620` `let now = chrono::Utc::now().naive_utc();`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:621` `sqlx::query("UPDATE mem_groups SET status = 'deleted', updated_at = ? WHERE group_id = ?")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:622` `.bind(now)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:623` `.bind(&group_id)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:624` `.execute(pool)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:625` `.await`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:636` `if let Ok(key_hash) = row.try_get::<String, _>("key_hash") {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:637` `state.api_key_cache.invalidate(&key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:638` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:639` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:640` `sqlx::query("UPDATE mem_api_keys SET is_active = 0 WHERE group_id = ? AND is_active = 1")`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:641` `.bind(&group_id)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:654` `"group_id": group_id,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:655` `"status": "deleted",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:656` `})))`
- `sources/memoria/memoria/crates/memoria-api/src/routes/groups.rs:657` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:88`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:87` `"memory_correct",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:88` `"memory_purge",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:89` `"memory_observe",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:90` `"memory_governance",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:91` `"memory_consolidate",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:92` `"memory_reflect",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:109` `tool,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:110` `"memory_store" | "memory_correct" | "memory_purge" | "memory_observe"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:111` `)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:112` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:113` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:114` `fn mcp_tool_mutates_main_regardless_checkout(tool: &str) -> bool {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:120` `match tool {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:121` `"memory_store" | "memory_correct" | "memory_purge" | "memory_observe" => {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:122` `Some(DirtyMask::MEMORY)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:123` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:124` `"memory_governance"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:125` `| "memory_consolidate"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:129` `"memory_feedback" => Some(DirtyMask::FEEDBACK),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:130` `"memory_snapshot" | "memory_snapshot_delete" => Some(DirtyMask::SNAPSHOT),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:131` `"memory_rollback" => Some(DirtyMask::FULL),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:132` `"memory_branch" | "memory_branch_delete" => Some(DirtyMask::BRANCH),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:133` `"memory_checkout" | "memory_merge" | "memory_pick" => Some(DirtyMask::FULL),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:134` `_ => None,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:135` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:136` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:152` `| "memory_correct"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:153` `| "memory_purge"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:154` `| "memory_observe"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:155` `| "memory_governance"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:156` `| "memory_rebuild_index"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:157` `| "memory_consolidate"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:163` `| "memory_snapshot"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:164` `| "memory_snapshot_delete"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:165` `| "memory_rollback"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:166` `| "memory_branch"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:167` `| "memory_checkout"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:168` `| "memory_merge"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:169` `| "memory_pick"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:170` `| "memory_branch_delete"`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:171` `| "memory_apply" => Some(SCOPE_MEMORY_WRITE),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:172` `_ => None,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:173` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:174` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:807` `// memoria-mcp and may change independently of this list.`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:808` `for name in &["memory_store", "memory_search", "memory_purge"] {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:809` `let params = json!({"name": name});`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:810` `assert_eq!(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:811` `tracking_path("tools/call", Some(&params)),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/mcp.rs:812` `format!("/mcp/{name}"),`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:588`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:587` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:588` `pub async fn delete_memory(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:589` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:590` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:591` `Path(id): Path<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:592` `Query(q): Query<BranchQuery>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:612` `.service`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:613` `.purge_on_branch(&effective_user_id, branch_param(q.branch.as_deref()), &id)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:614` `.await`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:615` `.map_err(api_err_typed)?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:616` `Ok(StatusCode::NO_CONTENT)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:617` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:618` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:619` `pub async fn purge_memories(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:620` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:621` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:622` `Json(req): Json<PurgeRequest>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:623` `) -> ApiResult<PurgeResponse> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:624` `let selector = req`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:625` `.selector()`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:626` `.map_err(|e| (StatusCode::UNPROCESSABLE_ENTITY, e))?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:627` `let branch = branch_param(req.branch.as_deref());`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:628` `let result = match selector {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:629` `PurgeSelector::MemoryIds(ids) => {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:630` `if !auth.is_master {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:631` `for id in &ids {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:632` `let mem = state`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:633` `.service`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:647` `.service`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:648` `.purge_batch_on_branch(auth.scope_id(), branch, &id_refs)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:649` `.await`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:650` `.map_err(api_err)?`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:651` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:652` `PurgeSelector::Topic(topic) => state`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:653` `.service`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:654` `.purge_by_topic_on_branch(auth.scope_id(), branch, &topic)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:655` `.await`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:656` `.map_err(api_err)?,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:657` `PurgeSelector::Session {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:658` `session_id,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:659` `memory_types,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:660` `} => state`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:661` `.service`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:662` `.purge_by_session_id_on_branch(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:663` `auth.scope_id(),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:664` `branch,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:665` `&session_id,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:666` `memory_types.as_deref(),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:669` `.map_err(api_err)?,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:670` `PurgeSelector::None => memoria_service::PurgeResult {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:671` `purged: 0,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:672` `snapshot_name: None,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:673` `warning: None,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:674` `},`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:675` `};`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:676` `Ok(Json(PurgeResponse {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:677` `purged: result.purged,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:678` `snapshot_name: result.snapshot_name,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:679` `warning: result.warning,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:680` `}))`
- `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:681` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:839`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:838` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:839` `pub async fn delete_snapshot(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:840` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:841` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:842` `Path(name): Path<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:843` `) -> Result<StatusCode, (StatusCode, String)> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:846` `auth.scope_id(),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:847` `"memory_snapshot_delete",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:848` `json!({ "names": name }),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:849` `)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:850` `.await?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:851` `Ok(StatusCode::NO_CONTENT)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:853` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:854` `pub async fn delete_snapshot_bulk(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:855` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:856` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:857` `Json(req): Json<serde_json::Value>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:858` `) -> Result<Json<serde_json::Value>, (StatusCode, String)> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:859` `let r = git_call(&state, auth.scope_id(), "memory_snapshot_delete", req).await?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:860` `Ok(Json(r))`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:861` `}`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:862` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:863` `pub async fn rollback(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1070` ``
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1071` `pub async fn delete_branch(`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1072` `State(state): State<AppState>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1073` `auth: AuthUser,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1074` `Path(name): Path<String>,`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1075` `) -> Result<StatusCode, (StatusCode, String)> {`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1078` `auth.scope_id(),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1079` `"memory_branch_delete",`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1080` `json!({ "name": name }),`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1081` `)`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1082` `.await?;`
- `sources/memoria/memoria/crates/memoria-api/src/routes/snapshots.rs:1083` `Ok(StatusCode::NO_CONTENT)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-api/src/state.rs:89`; body evidence.

- `sources/memoria/memoria/crates/memoria-api/src/state.rs:88` ``
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:89` `self.invalidate(key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:90` `None`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:91` `}`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:92` ``
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:93` `pub fn insert(&self, key_hash: String, principal: CachedApiKeyPrincipal) {`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:109` ``
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:110` `pub fn invalidate(&self, key_hash: &str) {`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:111` `if let Ok(mut cache) = self.inner.write() {`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:112` `cache.remove(key_hash);`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:113` `}`
- `sources/memoria/memoria/crates/memoria-api/src/state.rs:114` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:69`; body evidence.

- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:68` `if seed.is_outdated && !mid.is_empty() {`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:69` `let _ = self.purge_ids(&client, &[mid], "seed is_outdated");`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:70` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:71` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:72` `Err(e) => {`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:73` `exec.error = Some(format!("seed failed: {e}"));`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:199` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:200` `"purge" => {`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:201` `let mut body = json!({"reason": step.reason.as_deref().unwrap_or("benchmark")});`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:202` `if let Some(t) = &step.topic {`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:203` `body["topic"] = json!(t);`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:204` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:205` `client`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:206` `.post(format!("{}/v1/memories/purge", self.base_url))`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:207` `.json(&body)`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:208` `.send()?`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:209` `.error_for_status()?;`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:210` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:242` ``
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:243` `fn purge_ids(&self, client: &Client, ids: &[String], reason: &str) -> anyhow::Result<()> {`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:244` `client`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:245` `.post(format!("{}/v1/memories/purge", self.base_url))`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:246` `.json(&json!({"memory_ids": ids, "reason": reason}))`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:247` `.send()?`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:248` `.error_for_status()?;`
- `sources/memoria/memoria/crates/memoria-cli/src/benchmark_executor.rs:249` `Ok(())`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:258`; body evidence.

- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:257` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:258` `async fn delete(&self, path: &str, stats: &Stats, expected: &[u16]) -> u16 {`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:259` `let t0 = Instant::now();`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:260` `let res = self`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:261` `.client`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:262` `.delete(format!("{}{path}", self.base))`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:263` `.bearer_auth(&self.token)`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:264` `.header("X-Impersonate-User", &self.user_id)`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:265` `.send()`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:266` `.await;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:296` `let list = Arc::new(Stats::new("list"));`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:297` `let purge = Arc::new(Stats::new("purge"));`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:298` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:299` `let deadline = Instant::now() + duration;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:300` `while Instant::now() < deadline {`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:301` `// Phase 1: bootstrap , retrieve context (agent always does this first)`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:385` `c.post(`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:386` `"/v1/memories/purge",`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:387` `serde_json::json!({"topic": "load-test working", "reason": "session end"}),`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:388` `&purge,`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:389` `&[200],`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:390` `)`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:391` `.await;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:392` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:406` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:407` `vec![retrieve, store, search, correct, list, purge]`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:408` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:409` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:410` `// ── Scenario 2: Git-for-Data (snapshot/branch/merge) ────────────────────────`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:411` `//`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:422` `let snapshots_list = Arc::new(Stats::new("snapshots_list"));`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:423` `let branch_delete = Arc::new(Stats::new("branch_delete"));`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:424` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:425` `let deadline = Instant::now() + duration;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:426` `let mut iter = 0u32;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:427` `while Instant::now() < deadline {`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:502` `// cleanup branch`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:503` `c.delete(`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:504` `&format!("/v1/branches/{branch_name}"),`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:505` `&branch_delete,`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:506` `&[200, 204],`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:507` `)`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:508` `.await;`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:509` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:535` `snapshots_list,`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:536` `branch_delete,`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:537` `]`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:538` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:539` ``
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:540` `// ── Scenario 3: Maintenance (governance/consolidate/reflect + metrics) ──────`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:880` `}),`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:881` `("purge", {`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:882` `let (s, _) = c`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:883` `.post(`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:884` `"/v1/memories/purge",`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:885` `serde_json::json!({"topic":"preflight","reason":"cleanup"}),`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:886` `&d,`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:887` `&[200],`
- `sources/memoria/memoria/crates/memoria-cli/src/bin/loadtest.rs:888` `)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1459`; body evidence.

- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1458` `"memory_correct",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1459` `"memory_purge",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1460` `"memory_profile",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1461` `"memory_feedback",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1462` `"memory_capabilities",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1463` `"memory_governance",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1467` `"memory_snapshots",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1468` `"memory_snapshot_delete",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1469` `"memory_rollback",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1470` `"memory_branch",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1471` `"memory_branches",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1472` `"memory_checkout",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1473` `"memory_merge",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1474` `"memory_branch_delete",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1475` `"memory_diff",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1476` `"memory_count",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1477` `"memory_observe",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:1478` `"memory_id",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3153` `let tar = flate2::read::GzDecoder::new(tar_gz);`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3154` `let mut archive = tar::Archive::new(tar);`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3155` `archive.unpack(tmp.path()).unwrap();`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3156` ``
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3157` `let current_exe = std::env::current_exe().unwrap();`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3158` `let new_bin = tmp.path().join("memoria");`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3159` `if let Err(e) = self_replace::self_replace(&new_bin) {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3541` ``
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3542` `async fn soft_delete(&self, _: &str) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3543` `Ok(())`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3544` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3545` ``
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3546` `async fn list_active(&self, _: &str, _: i64) -> Result<Vec<Memory>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3712` `let err = run_with_edit_log_drain(service.clone(), async {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3713` `service.send_edit_log("u1", "purge", Some("m1"), None, "test", None);`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3714` `Err::<(), _>(anyhow::anyhow!("boom"))`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3715` `})`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3716` `.await`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3717` `.expect_err("helper should return original error");`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3722` `assert_eq!(drained.len(), 1);`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3723` `assert_eq!(drained[0].operation, "purge");`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3724` `}`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3725` ``
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3726` `#[test]`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3727` `fn migrate_cli_defaults_to_dry_run() {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3905` `"memory_search",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3906` `"memory_purge",`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3907` `] {`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3908` `assert!(`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3909` `approved.iter().any(|v| v.as_str() == Some(tool)),`
- `sources/memoria/memoria/crates/memoria-cli/src/main.rs:3910` `"autoApprove is missing tool: {tool}"`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:16`; body evidence.

- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:15` `async fn update(&self, memory: &Memory) -> Result<(), MemoriaError>;`
- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:16` `async fn soft_delete(&self, memory_id: &str) -> Result<(), MemoriaError>;`
- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:17` `async fn list_active(&self, user_id: &str, limit: i64) -> Result<Vec<Memory>, MemoriaError>;`
- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:18` `async fn search_fulltext(`
- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:19` `&self,`
- `sources/memoria/memoria/crates/memoria-core/src/interfaces.rs:20` `user_id: &str,`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:21`; body evidence.

- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:20` ``
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:21` `fn delete_target(args: &[String]) -> Result<Option<&str>, &'static str> {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:22` `match args {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:23` `[] => Ok(None),`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:24` `[flag, name, offline] if flag == "--delete" && offline == "--all-writers-stopped" => {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:25` `if candidate(name) {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:26` `Ok(Some(name))`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:27` `} else {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:28` `Err("Refusing unexpected table name; use an exact generated stage/branch name")`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:30` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:31` `_ => Err("Usage: restore_cleanup [--delete EXACT_TABLE --all-writers-stopped]"),`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:32` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:33` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:34` ``
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:35` `async fn registered(pool: &MySqlPool, table: &str) -> Result<bool, sqlx::Error> {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:44` ``
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:45` `async fn delete_unregistered(`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:46` `pool: &MySqlPool,`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:47` `table: &str,`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:48` `) -> Result<(), Box<dyn std::error::Error>> {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:49` `if !candidate(table) {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:52` `if registered(pool, table).await? {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:53` `return Err("Refusing to delete a registered branch".into());`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:54` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:55` `let mut ddl = QueryBuilder::<MySql>::new(if table.starts_with("br_") {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:56` `"DATA BRANCH DELETE TABLE "`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:57` `} else {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:58` `"DROP TABLE "`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:59` `});`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:60` `ddl.push("'").push(table).push("'");`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:67` `let args: Vec<String> = std::env::args().skip(1).collect();`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:68` `let delete = delete_target(&args)?;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:69` `let pool = MySqlPool::connect(&std::env::var("DATABASE_URL")?).await?;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:70` `let database: String = sqlx::query_scalar("SELECT DATABASE()")`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:71` `.fetch_one(&pool)`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:72` `.await?;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:84` `.await?;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:85` `if let Some(table) = delete {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:86` `if !tables.iter().any(|t| t == table) {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:87` `return Err("Target table does not exist in the selected database".into());`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:88` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:89` `// The operator's offline confirmation is essential: absence from the`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:91` `// This tool cannot fence other processes and is NEVER a scheduled reaper.`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:92` `delete_unregistered(&pool, table).await?;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:93` `println!("Deleted {database}.{table}. No automatic undo; recovery requires a retained backup/snapshot.");`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:94` `} else {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:95` `println!("Read-only inspection of {database}. Candidates may still be in use; stop ALL writers before deletion.");`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:96` `for table in tables.into_iter().filter(|t| candidate(t)) {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:97` `let state = if registered(&pool, &table).await? {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:98` `"protected: registered branch"`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:99` `} else {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:113` `#[test]`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:114` `fn deletion_requires_exact_target_and_offline_confirmation() {`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:115` `assert_eq!(delete_target(&[]), Ok(None));`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:116` `let name = "mem_restore_0123456789abcdef0123456789abcdef";`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:117` `assert!(delete_target(&["--delete".into(), name.into()]).is_err());`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:118` `assert_eq!(`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:119` `delete_target(&[`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:120` `"--delete".into(),`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:121` `name.into(),`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:122` `"--all-writers-stopped".into()`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:123` `]),`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:124` `Ok(Some(name))`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:189` `}`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:190` `let protected = delete_unregistered(&pool, "br_12345678_registered").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:191` `let unicode_protected = delete_unregistered(&pool, "br_12345678_已有").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:192` `let unicode_orphan = delete_unregistered(&pool, "br_12345678_孤儿").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:193` `let unexpected = delete_unregistered(&pool, "mem_branches").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:194` `let orphan = delete_unregistered(&pool, "br_12345678_orphan").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:195` `let stage =`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:196` `delete_unregistered(&pool, "mem_restore_0123456789abcdef0123456789abcdef").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:197` `sqlx::raw_sql("DROP TABLE mem_branches; CREATE TABLE br_12345678_unverified (id INT)")`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:198` `.execute(&pool)`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:199` `.await`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:200` `.unwrap();`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:201` `let closed = delete_unregistered(&pool, "br_12345678_unverified").await;`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:202` `let tables: Vec<String> = sqlx::query_scalar("SELECT table_name FROM information_schema.tables WHERE table_schema = ? ORDER BY table_name")`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:203` `.bind(&db).fetch_all(&pool).await.unwrap();`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:204` `let source_rows: Vec<i32> = sqlx::query_scalar("SELECT id FROM cleanup_source ORDER BY id")`
- `sources/memoria/memoria/crates/memoria-git/examples/restore_cleanup.rs:205` `.fetch_all(&pool)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-git/src/service.rs:73`; body evidence.

- `sources/memoria/memoria/crates/memoria-git/src/service.rs:72` `pool: &MySqlPool,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:73` `delete: &str,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:74` `insert: &str,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:75` `) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:76` `let mut tx = pool.begin().await.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:77` `if let Err(error) = sqlx::query(delete).execute(&mut *tx).await {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:78` `return Err(restore_statement_error(error, tx.rollback().await));`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:79` `}`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:80` `if let Err(error) = sqlx::query(insert).execute(&mut *tx).await {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:81` `return Err(restore_statement_error(error, tx.rollback().await));`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:119` `.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:120` `let mut delete = QueryBuilder::<MySql>::new("DELETE FROM ");`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:121` `delete.push(&target_table);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:122` `let mut insert = QueryBuilder::<MySql>::new("INSERT INTO ");`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:123` `insert`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:124` `.push(&target_table)`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:125` `.push(" (")`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:130` `.push(&stage_table);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:131` `replace_from_stage(&pool, &delete.into_sql(), &insert.into_sql()).await`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:132` `}`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:133` ``
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:134` `/// Look up a column index by name, case-insensitively.`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:135` `///`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:253` `pub source: String, // branch table name or main table name`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:254` `pub flag: String,   // INSERT | UPDATE | DELETE`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:255` `pub memory_id: String,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:256` `pub content: String,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:257` `pub memory_type: String,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:258` `pub is_active: i8,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:907` `let db = quote_identifier(&self.db_name);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:908` `exec_ddl(&self.pool, &format!("data branch delete table {db}.{safe}")).await`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:909` `}`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:910` ``
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:911` `/// Native branch merge: 'data branch merge {branch} into {main} when conflict skip'.`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:912` `/// Inserts rows from branch that don't exist in main (by PK). Account-level but`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1332` `.join(",");`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1333` `let delete_sql = format!(`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1334` `"DELETE FROM {main_table_ref} WHERE user_id = ? AND memory_id IN ({ph})"`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1335` `);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1336` `let mut q = sqlx::query(&delete_sql).bind(user_id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1337` `for id in &restore_adds {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1338` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1339` `}`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1340` `q.execute(&mut *tx).await.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1395` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1396` `"DELETE FROM {main_table_ref} WHERE user_id = ? AND memory_id = ?"`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1397` `))`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1398` `.bind(user_id)`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1399` `.bind(&pair.old_id)`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1400` `.execute(&mut *tx)`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1495` `let del_sql = format!(`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1496` `"DELETE FROM {main_table_ref} WHERE user_id = ? AND memory_id IN ({ph})"`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1497` `);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1498` `let mut q = sqlx::query(&del_sql).bind(user_id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1499` `for id in &result.applied_removes {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1500` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1586` `let del_sql = format!(`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1587` `"DELETE FROM {main_table_ref} WHERE user_id = ? AND memory_id IN ({main_ph})"`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1588` `);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1589` `let mut q = sqlx::query(&del_sql).bind(user_id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1590` `for id in &main_ids {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1591` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1712` `&git.pool,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1713` `"DELETE FROM target",`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1714` `"INSERT INTO target SELECT * FROM stage",`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1715` `));`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1716` `assert_send(git.restore_table_from_snapshot("memories", "snapshot"));`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1717` `};`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1746` `#[ignore = "requires a disposable MatrixOne server via DATABASE_URL"]`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1747` `async fn failed_insert_rolls_back_preceding_delete() {`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1748` `let url = std::env::var("DATABASE_URL").expect("DATABASE_URL required");`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1749` `let options: sqlx::mysql::MySqlConnectOptions = url.parse().unwrap();`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1750` `let admin = MySqlPool::connect_with(options.clone().database("mo_catalog"))`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1751` `.await`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1765` `&pool,`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1766` `"DELETE FROM target",`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1767` `"INSERT INTO target SELECT * FROM stage",`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1768` `)`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1769` `.await;`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1770` `let rows: Vec<(i32, String)> = sqlx::query_as("SELECT id, content FROM target")`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1788` `vec![(1, "current".into())],`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1789` `"DELETE must be rolled back with the failed INSERT"`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1790` `);`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1791` `}`
- `sources/memoria/memoria/crates/memoria-git/src/service.rs:1792` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:574`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:573` `{`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:574` `"name": "memory_snapshot_delete",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:575` `"description": "Delete snapshots by name(s), prefix, or age",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:576` `"inputSchema": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:577` `"type": "object",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:578` `"properties": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:579` `"names": {"type": "string"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:714` `{`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:715` `"name": "memory_branch_delete",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:716` `"description": "Delete a memory branch",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:717` `"inputSchema": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:718` `"type": "object",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:719` `"properties": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:720` `"name": {"type": "string"}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:766` `"default": [],`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:767` `"description": "memory_ids to remove (soft-delete from main)"`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:768` `},`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:769` `"accept_branch_conflicts": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:770` `"type": "array",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:771` `"items": {"type": "string"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:784` `MemorySnapshots,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:785` `MemorySnapshotDelete,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:786` `MemoryRollback,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:787` `MemoryBranch,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:788` `MemoryBranches,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:789` `MemoryCheckout,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:791` `MemoryPick,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:792` `MemoryBranchDelete,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:793` `MemoryDiff,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:794` `MemoryApply,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:795` `Unknown(String),`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:796` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:807` `"memory_snapshots" => GitToolCallName::MemorySnapshots,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:808` `"memory_snapshot_delete" => GitToolCallName::MemorySnapshotDelete,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:809` `"memory_rollback" => GitToolCallName::MemoryRollback,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:810` `"memory_branch" => GitToolCallName::MemoryBranch,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:811` `"memory_branches" => GitToolCallName::MemoryBranches,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:812` `"memory_checkout" => GitToolCallName::MemoryCheckout,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:814` `"memory_pick" => GitToolCallName::MemoryPick,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:815` `"memory_branch_delete" => GitToolCallName::MemoryBranchDelete,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:816` `"memory_diff" => GitToolCallName::MemoryDiff,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:817` `"memory_apply" => GitToolCallName::MemoryApply,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:818` `_ => GitToolCallName::Unknown(name.to_string()),`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:819` `};`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:828` `return Ok(crate::tool_result::error(format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:829` `"Snapshot limit reached ({MAX_USER_SNAPSHOTS}) for user {user_id}. Delete old snapshots first."`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:830` `)));`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:831` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:832` `let snap_name = args["name"].as_str().unwrap_or("");`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:833` `let sql = snapshot_store(svc, user_id).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:938` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:939` `GitToolCallName::MemorySnapshotDelete => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:940` `let sql = snapshot_store(svc, user_id).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:941` `let git = git_for_store(&sql)?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:942` `let snaps = visible_snapshots_for_user(svc, user_id).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:943` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:944` `let to_delete: Vec<VisibleSnapshot> = if let Some(names) = args["names"].as_str() {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:945` `let name_set: HashSet<String> =`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:946` `names.split(',').map(|n| n.trim().to_string()).collect();`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:947` `snaps`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:948` `.iter()`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:991` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:992` `let count = to_delete.len();`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:993` `for snapshot in &to_delete {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:994` `git.drop_snapshot(&snapshot.internal_name)`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:995` `.await`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:996` `.map_err(git_err)?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:997` `if snapshot.registered {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1001` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1002` `let display: Vec<_> = to_delete.iter().map(|s| s.display_name.clone()).collect();`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1003` `Ok(mcp_text(&format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1004` `"Deleted {count} snapshot(s): {}",`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1005` `display.join(", ")`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1006` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1007` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1008` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1022` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1023` `sql.invalidate_user_caches(user_id).await;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1024` `Ok(mcp_text(&format!("Rolled back to snapshot '{snap_name}'")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1025` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1026` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1027` `GitToolCallName::MemoryBranch => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1065` `return Ok(crate::tool_result::error(format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1066` `"Branch limit reached ({MAX_BRANCHES}). Delete old branches first."`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1067` `)));`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1068` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1069` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1070` `// Duplicate name check , only reject if an active branch with same name exists`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1578` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1579` `GitToolCallName::MemoryBranchDelete => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1580` `let branch = args["name"].as_str().unwrap_or("");`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1581` `if branch == "main" {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1582` `return Ok(crate::tool_result::error("Cannot delete main"));`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1583` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1584` `let sql = svc.user_sql_store(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1585` `let git = git_for_store(&sql)?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1586` `let branches = sql.list_branches(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1593` `sql.deregister_branch(user_id, branch).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1594` `Ok(mcp_text(&format!("Deleted branch '{branch}'")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1595` `} else {`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1596` `Ok(crate::tool_result::classified_error(crate::tool_result::ErrorKind::Rejected,`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1597` `format!("Branch '{branch}' not found. List branches and check the name before retrying.")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/git_tools.rs:1598` `}`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:3`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:2` `pub mod git_tools;`
- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:3` `pub mod purge_args;`
- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:4` `pub mod remote;`
- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:5` `mod server;`
- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:6` `pub mod tool_result;`
- `sources/memoria/memoria/crates/memoria-mcp/src/lib.rs:7` `pub mod tools;`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:7`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:6` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:7` `pub(crate) struct MemoryPurgeArgs {`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:8` `pub(crate) memory_id: Option<String>,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:9` `pub(crate) topic: Option<String>,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:10` `pub(crate) session_id: Option<String>,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:11` `pub(crate) memory_types: Option<Vec<MemoryType>>,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:14` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:15` `pub(crate) fn parse_memory_purge_args(args: &Value) -> Result<MemoryPurgeArgs> {`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:16` `let memory_id = args["memory_id"]`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:17` `.as_str()`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:18` `.map(str::trim)`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:19` `.filter(|s| !s.is_empty())`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:62` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:63` `Ok(MemoryPurgeArgs {`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:64` `memory_id,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:65` `topic,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:66` `session_id,`
- `sources/memoria/memoria/crates/memoria-mcp/src/purge_args.rs:67` `memory_types,`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:4`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:3` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:4` `use crate::purge_args::parse_memory_purge_args;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:5` `use anyhow::Result;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:6` `use reqwest::Client;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:7` `use serde_json::{json, Value};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:8` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:447` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:448` `"memory_purge" => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:449` `let purge_args = parse_memory_purge_args(&args)?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:450` `if let Some(memory_id) = purge_args.memory_id {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:451` `let ids: Vec<&str> = memory_id`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:452` `.split(',')`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:453` `.map(str::trim)`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:454` `.filter(|s| !s.is_empty())`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:457` `.client`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:458` `.post(self.url("/v1/memories/purge"))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:459` `.json(&json!({"memory_ids": ids, "branch": purge_args.branch}))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:460` `.send()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:461` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:462` `let body = Self::parse_response(r).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:463` `let purged = Self::require_i64(&body, "memory_purge", "purged")?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:464` `Ok(Self::mcp_text(&format!("Purged {purged} memory(s)")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:465` `} else if let Some(topic) = purge_args.topic {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:466` `let r = self`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:467` `.client`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:468` `.post(self.url("/v1/memories/purge"))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:469` `.json(&json!({"topic": topic, "branch": purge_args.branch}))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:470` `.send()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:471` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:472` `let body = Self::parse_response(r).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:473` `let purged = Self::require_i64(&body, "memory_purge", "purged")?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:474` `Ok(Self::mcp_text(&format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:475` `"Purged {purged} memory(s) matching '{topic}'"`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:476` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:477` `} else if let Some(session_id) = purge_args.session_id {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:478` `let r = self`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:479` `.client`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:480` `.post(self.url("/v1/memories/purge"))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:481` `.json(&json!({`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:482` `"session_id": session_id,`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:483` `"memory_types": purge_args.memory_types.map(|memory_types| {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:484` `memory_types`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:485` `.into_iter()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:486` `.map(|memory_type| memory_type.to_string())`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:487` `.collect::<Vec<_>>()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:488` `}),`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:489` `"branch": purge_args.branch,`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:490` `}))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:491` `.send()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:492` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:493` `let body = Self::parse_response(r).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:494` `let purged = Self::require_i64(&body, "memory_purge", "purged")?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:495` `Ok(Self::mcp_text(&format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:496` `"Purged {purged} memory(s) for session '{session_id}'"`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:497` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:498` `} else {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:499` `Ok(crate::tool_result::error(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:500` `"Provide memory_id, topic, or session_id",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:819` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:820` `"memory_snapshot_delete" => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:821` `// names can be a single string or array`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:822` `let mut payload = args.clone();`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:823` `if let Some(names_str) = args["names"].as_str() {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:824` `// Convert comma-separated string to array`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:829` `.client`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:830` `.post(self.url("/v1/snapshots/delete"))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:831` `.json(&payload)`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:832` `.send()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:833` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:834` `let body = Self::parse_response(r).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:837` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:838` `let deleted = Self::require_i64(&body, "memory_snapshot_delete", "deleted")?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:839` `Ok(Self::mcp_text(&format!("Deleted {deleted} snapshot(s).")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:840` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:841` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:842` `"memory_rollback" => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:843` `let name = args["name"].as_str().unwrap_or("");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:980` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:981` `"memory_branch_delete" => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:982` `let name = args["name"].as_str().unwrap_or("");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:983` `let r = self`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:984` `.client`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:985` `.delete(self.url(&format!("/v1/branches/{name}")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:986` `.send()`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:987` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:988` `let _body = Self::parse_response_allow_empty(r).await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:989` `Ok(Self::mcp_text(&format!("Branch '{name}' deleted.")))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:990` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:991` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:992` `"memory_observe" => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:993` `let mut payload = json!({`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1048` `extract::{Path, State},`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1049` `routing::{delete, post},`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1050` `Json, Router,`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1051` `};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1052` `use serde_json::{json, Value};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1053` `use std::sync::{Arc, Mutex};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1143` `#[tokio::test]`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1144` `async fn remote_delete_and_rebuild_failures_are_not_successes() {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1145` `use crate::tool_result::{error_kind, execution_error, is_error, ErrorKind};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1146` `use axum::http::{header::CONTENT_TYPE, StatusCode};`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1147` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1148` `let app = Router::new().route(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1149` `"/v1/branches/:name",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1150` `delete(|| async {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1151` `(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1152` `StatusCode::CONFLICT,`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1153` `[(CONTENT_TYPE, "text/plain")],`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1154` `"Branch 'missing' not found",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1160` `let error = remote`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1161` `.call("memory_branch_delete", json!({"name": "missing"}))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1162` `.await`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1163` `.expect_err("a 409 must not be reported as a deleted branch");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1164` `let result = execution_error("memory_branch_delete", error);`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1165` `assert!(is_error(&result));`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1166` `assert_eq!(error_kind(&result), Some(ErrorKind::Rejected));`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1167` `let text = result["content"][0]["text"].as_str().unwrap();`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1168` `assert!(text.contains("Branch 'missing' not found"), "{text}");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1169` `assert!(!text.contains("deleted"), "{text}");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1170` `server.abort();`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1171` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1172` `let app = Router::new().route(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1173` `"/v1/governance",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1198` `#[tokio::test]`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1199` `async fn remote_branch_delete_accepts_empty_success() {`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1200` `use axum::http::StatusCode;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1201` `let app = Router::new().route(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1202` `"/v1/branches/:name",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1203` `delete(|| async { StatusCode::NO_CONTENT }),`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1204` `);`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1205` `let (base, server) = serve(app).await;`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1206` `let remote = RemoteClient::new(&base, None, "test".into(), None);`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1207` `let result = remote`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1208` `.call("memory_branch_delete", json!({"name": "topic"}))`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1209` `.await`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1210` `.expect("204 is a successful delete");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1211` `assert_eq!(result["content"][0]["text"], "Branch 'topic' deleted.");`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1212` `assert!(result.get("isError").is_none());`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1213` `server.abort();`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1214` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1215` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1314` `(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1315` `"/v1/memories/purge",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1316` `StatusCode::OK,`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1317` `"{}",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1318` `"memory_purge",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1319` `json!({"memory_id": "mem_1"}),`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1320` `"Purged",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1321` `),`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1322` `(`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1323` `"/v1/memories/retrieve",`
- `sources/memoria/memoria/crates/memoria-mcp/src/remote.rs:1324` `StatusCode::OK,`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:96`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:95` `"memory_snapshots",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:96` `"memory_snapshot_delete",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:97` `"memory_rollback",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:98` `"memory_branch",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:99` `"memory_branches",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:100` `"memory_checkout",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:104` `"memory_apply",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:105` `"memory_branch_delete",`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:106` `];`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:107` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:108` `fn is_git_tool(name: &str) -> bool {`
- `sources/memoria/memoria/crates/memoria-mcp/src/server.rs:109` `GIT_TOOL_NAMES.contains(&name)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:4`; body evidence.

- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:3` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:4` `use crate::purge_args::parse_memory_purge_args;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:5` `use crate::tool_result::error as mcp_error;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:6` `use anyhow::{Context, Result};`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:7` `use memoria_core::{MemoryType, TrustTier};`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:8` `use memoria_git::GitForDataService;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:149` `MemoryCorrect,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:150` `MemoryPurge,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:151` `MemoryProfile,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:152` `MemoryList,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:153` `MemoryCapabilities,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:154` `MemoryGovernance,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:184` `"Available tools: memory_store, memory_retrieve, memory_search, ",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:185` `"memory_correct, memory_purge, memory_profile, memory_list, ",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:186` `"memory_capabilities, memory_governance, memory_consolidate, ",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:187` `"memory_reflect, memory_feedback",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:188` `"\n\nmemory_store trust_tier guide:",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:189` `"\n- T1 (Verified): directly stated or explicitly confirmed facts, preferences, or decisions.",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:274` `{`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:275` `"name": "memory_purge",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:276` `"description": "Delete memories by ID, by topic keyword, or by exact session_id with optional memory type filtering",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:277` `"inputSchema": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:278` `"type": "object",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:279` `"properties": {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:280` `"memory_id": {"type": "string", "description": "Single ID or comma-separated batch"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:281` `"topic": {"type": "string", "description": "Keyword , bulk-delete all matching memories"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:282` `"session_id": {"type": "string", "description": "Exact session identifier , bulk-delete memories from that session"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:283` `"memory_types": {"type": "array", "items": {"type": "string", "enum": ["semantic", "working", "episodic", "profile", "tool_result", "procedural"]}, "description": "Optional memory type filter. Only valid with session_id"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:284` `"branch": {"type": "string", "description": "Optional branch to purge from without changing the active checkout"},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:285` `"reason": {"type": "string"}`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:286` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:287` `}`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:288` `},`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:385` `"memory_correct" => ToolCallName::MemoryCorrect,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:386` `"memory_purge" => ToolCallName::MemoryPurge,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:387` `"memory_profile" => ToolCallName::MemoryProfile,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:388` `"memory_list" => ToolCallName::MemoryList,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:389` `"memory_capabilities" => ToolCallName::MemoryCapabilities,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:390` `"memory_governance" => ToolCallName::MemoryGovernance,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:627` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:628` `ToolCallName::MemoryPurge => {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:629` `let purge_args = parse_memory_purge_args(&args)?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:630` `if let Some(memory_id) = purge_args.memory_id {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:631` `// Batch: comma-separated IDs`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:632` `let ids: Vec<&str> = memory_id`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:633` `.split(',')`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:634` `.map(str::trim)`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:637` `let result = service`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:638` `.purge_batch_on_branch(user_id, branch_arg(&args), &ids)`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:639` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:640` `Ok(mcp_text(&format_purge_msg(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:641` `&format!("Purged {} memory(s)", result.purged),`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:642` `&result,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:643` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:644` `} else if let Some(topic) = purge_args.topic {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:645` `// Bulk by keyword: exact text match then purge`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:646` `let result = service`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:647` `.purge_by_topic_on_branch(user_id, branch_arg(&args), &topic)`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:648` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:649` `Ok(mcp_text(&format_purge_msg(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:650` `&format!("Purged {} memory(s) matching '{topic}'", result.purged),`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:651` `&result,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:652` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:653` `} else if let Some(session_id) = purge_args.session_id {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:654` `let result = service`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:655` `.purge_by_session_id_on_branch(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:656` `user_id,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:657` `branch_arg(&args),`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:658` `&session_id,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:659` `purge_args.memory_types.as_deref(),`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:660` `)`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:661` `.await?;`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:662` `Ok(mcp_text(&format_purge_msg(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:663` `&format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:664` `"Purged {} memory(s) for session '{session_id}'",`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:665` `result.purged`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:666` `),`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:667` `&result,`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:668` `)))`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:669` `} else {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1330` ``
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1331` `fn format_purge_msg(base: &str, result: &memoria_service::PurgeResult) -> String {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1332` `let mut msg = base.to_string();`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1333` `if let Some(snap) = &result.snapshot_name {`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1334` `msg.push_str(&format!(`
- `sources/memoria/memoria/crates/memoria-mcp/src/tools.rs:1335` `"\nSafety snapshot: {snap} (use memory_rollback to undo)"`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:78`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:77` `// Clean up expired lock`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:78` `sqlx::query("DELETE FROM mem_distributed_locks WHERE lock_key = ? AND expires_at < NOW()")`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:79` `.bind(key)`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:80` `.execute(self.pool())`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:81` `.await`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:82` `.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:143` `async fn release(&self, key: &str, holder: &str) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:144` `sqlx::query("DELETE FROM mem_distributed_locks WHERE lock_key = ? AND holder_id = ?")`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:145` `.bind(key)`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:146` `.bind(holder)`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:147` `.execute(self.pool())`
- `sources/memoria/memoria/crates/memoria-service/src/distributed.rs:148` `.await`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/governance.rs:34`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:33` `pub async_tasks_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:34` `pub archived_working: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:35` `pub stale_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:36` `pub redundant_compressed: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:37` `pub orphaned_incrementals_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:38` `pub vector_index_rows: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:70` `/// Archive stale working memories and return per-user affected counts.`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:71` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:72` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:73` `stale_hours: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:74` `) -> Result<Vec<(String, i64)>, MemoriaError>;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:75` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:233` `let result = sqlx::query(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:234` `"DELETE FROM mem_async_tasks WHERE status IN ('completed', 'failed') \`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:235` `AND updated_at < DATE_SUB(NOW(), INTERVAL ? HOUR)",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:236` `)`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:237` `.bind(ttl_hours)`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:238` `.execute(self.pool())`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:243` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:244` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:245` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:246` `stale_hours: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:247` `) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:248` `let store = self.clone();`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:251` `for (_, store) in user_stores {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:252` `results.extend(SqlMemoryStore::archive_stale_working(&store, stale_hours).await?);`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:253` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:254` `return Ok(results);`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:255` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:256` `SqlMemoryStore::archive_stale_working(&store, stale_hours).await`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:257` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:258` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:259` `async fn cleanup_stale(&self, user_id: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:260` `let store = self.clone();`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:663` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:664` `async fn archive_stale_working_operation(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:665` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:666` `store: &dyn GovernanceStore,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:667` `state: &mut ExecutionState,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:668` `) {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:669` `match store.archive_stale_working(Self::STALE_WORKING_HOURS).await {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:670` `Ok(per_user) => {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:671` `let mut affected_users = Vec::new();`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:672` `for (user_id, count) in per_user {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:673` `if count > 0 {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:676` `&user_id,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:677` `"governance:archive_working",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:678` `None,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:679` `Some(&format!(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:680` `"{{\"archived\":{count},\"threshold_hours\":{}}}",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:681` `Self::STALE_WORKING_HOURS`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:682` `)),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:683` `&format!(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:684` `"archived {count} stale working memories (>{}h)",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:685` `Self::STALE_WORKING_HOURS`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:686` `),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:687` `state.snapshot_before.as_deref(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:688` `)`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:690` `affected_users.push(user_id);`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:691` `state.summary.archived_working += count;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:692` `state.summary.total_cleaned += count;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:693` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:694` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:695` `state.decisions.push(governance_decision(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:696` `GovernanceTask::Hourly,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:697` `"archive_stale_working",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:698` `format!(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:699` `"Archived {} stale working memories across {} users",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:700` `state.summary.archived_working,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:701` `affected_users.len()`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:702` `),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:703` `Some(if state.summary.archived_working > 0 {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:704` `1.0`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:705` `} else {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:706` `0.0`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:707` `}),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:721` `None,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:722` `"archive_stale_working",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:723` `&err,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:724` `&mut state.warnings,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:725` `),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:726` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1151` `match store.cleanup_edit_log(Self::EDIT_LOG_RETAIN_DAYS).await {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1152` `Ok(deleted) => {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1153` `state.decisions.push(governance_decision(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1154` `GovernanceTask::Weekly,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1155` `"cleanup_edit_log",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1156` `format!(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1157` `"Deleted {deleted} audit-log rows older than {} days",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1158` `Self::EDIT_LOG_RETAIN_DAYS`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1159` `),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1160` `Some(if deleted > 0 { 1.0 } else { 0.0 }),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1161` `vec![],`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1162` `state.snapshot_before.as_deref(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1163` `));`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1164` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1180` `match store.cleanup_feedback(Self::FEEDBACK_RETAIN_DAYS).await {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1181` `Ok(deleted) => {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1182` `state.decisions.push(governance_decision(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1183` `GovernanceTask::Weekly,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1184` `"cleanup_feedback",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1185` `format!(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1186` `"Deleted {deleted} feedback rows older than {} days",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1187` `Self::FEEDBACK_RETAIN_DAYS`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1188` `),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1189` `Some(if deleted > 0 { 1.0 } else { 0.0 }),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1190` `vec![],`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1191` `state.snapshot_before.as_deref(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1192` `));`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1193` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1244` `.await;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1245` `self.archive_stale_working_operation(store, state).await;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1246` `self.cleanup_stale_operation(GovernanceTask::Hourly, store, &plan.users, state)`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1247` `.await;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1248` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1249` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1322` `"cleanup_tool_results",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1323` `"archive_stale_working",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1324` `"cleanup_stale",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1325` `],`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1326` `GovernanceTask::Daily => &[`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1327` `"quarantine_low_confidence",`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1413` `metrics.insert(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1414` `"governance.archived_count".to_string(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1415` `state.summary.archived_working as f64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1416` `);`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1417` `metrics.insert(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1418` `"governance.stale_cleaned".to_string(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1419` `state.summary.stale_cleaned as f64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1564` `cleanup_tool_results_result: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1565` `archive_stale_working_result: Vec<(String, i64)>,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1566` `cleanup_stale_result: HashMap<String, i64>,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1567` `quarantine_result: HashMap<String, i64>,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1568` `compress_redundant_result: HashMap<String, i64>,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1569` `cleanup_orphaned_incrementals_result: HashMap<String, i64>,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1646` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1647` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1648` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1649` `stale_hours: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1650` `) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1651` `self.record(format!("archive_stale_working:{stale_hours}"));`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1652` `self.fail_if_requested("archive_stale_working")?;`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1653` `Ok(self.archive_stale_working_result.clone())`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1654` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1655` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1656` `async fn cleanup_stale(&self, user_id: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1657` `self.record(format!("cleanup_stale:{user_id}"));`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1768` `cleanup_tool_results_result: 4,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1769` `archive_stale_working_result: vec![("u1".into(), 2), ("u2".into(), 1)],`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1770` `cleanup_stale_result: HashMap::from([("u1".into(), 3), ("u2".into(), 0)]),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1771` `snapshot_result: Some("mem_snap_pre_hourly_deadbeef".into()),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1772` `..RecordingStore::default()`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1773` `};`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1787` `async_tasks_cleaned: 0,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1788` `archived_working: 3,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1789` `stale_cleaned: 3,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1790` `redundant_compressed: 0,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1791` `orphaned_incrementals_cleaned: 0,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1792` `vector_index_rows: 0,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1808` `"u1".into(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1809` `"governance:archive_working".into(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1810` `"archived 2 stale working memories (>24h)".into(),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1811` `Some("mem_snap_pre_hourly_deadbeef".into()),`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1812` `)));`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1813` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1814` ``
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1996` `}`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1997` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1998` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:1999` `_: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:2000` `) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/governance.rs:2001` `Ok(vec![])`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/lib.rs:49`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:48` `CandidateScore, ExplainLevel, FulltextSearchOptions, InMemoryFlusher, ListActiveOptions,`
- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:49` `MemoryService, PurgeResult, RetrievalExplain, RetrieveOptions, SessionScope,`
- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:50` `StructuredQueryOptions,`
- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:51` `ENTITY_EXTRACTION_DROPS,`
- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:52` `};`
- `sources/memoria/memoria/crates/memoria-service/src/lib.rs:53` `pub use stats_reporter::StatsReporter;`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:87`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:86` `tool_results_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:87` `archived_working: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:88` `stale_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:89` `redundant_compressed: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:90` `orphaned_incrementals_cleaned: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:91` `vector_index_rows: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:102` `tool_results_cleaned: value.tool_results_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:103` `archived_working: value.archived_working,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:104` `stale_cleaned: value.stale_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:105` `redundant_compressed: value.redundant_compressed,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:106` `orphaned_incrementals_cleaned: value.orphaned_incrementals_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/governance_hook.rs:107` `vector_index_rows: value.vector_index_rows,`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:216`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:215` `}`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:216` `async fn archive_stale_working(&self, _: i64) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:217` `Ok(vec![])`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:218` `}`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:219` `async fn cleanup_stale(&self, _: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/grpc_runtime.rs:220` `Ok(0)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:623`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:622` `&dir,`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:623` `"memoria-governance-stale-archive",`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:624` `"governance.plan",`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:625` `"fn memoria_plugin(ctx) { #{ requires_approval: false } }",`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:626` `);`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:627` ``
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:628` `let package = load_plugin_package(&dir, &HostPluginPolicy::development()).unwrap();`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:629` `assert_eq!(package.plugin_key, "governance:stale-archive:v1");`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:630` ``
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:631` `let _ = fs::remove_dir_all(dir);`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:632` `}`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/manifest.rs:633` ``

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:290`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:289` `}`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:290` `async fn archive_stale_working(&self, _: i64) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:291` `Ok(vec![])`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:292` `}`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:293` `async fn cleanup_stale(&self, _: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/plugin/rhai_runtime.rs:294` `Ok(0)`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:546`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:545` `async_tasks_cleaned = execution.summary.async_tasks_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:546` `archived_working = execution.summary.archived_working,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:547` `stale_cleaned = execution.summary.stale_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:548` `redundant_compressed = execution.summary.redundant_compressed,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:549` `orphaned_incrementals_cleaned = execution.summary.orphaned_incrementals_cleaned,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:550` `vector_index_rows = execution.summary.vector_index_rows,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:658` `}`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:659` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:660` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:661` `_: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:662` `) -> Result<Vec<(String, i64)>, crate::MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:663` `Ok(vec![])`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:735` `}`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:736` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:737` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:738` `_: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:739` `) -> Result<Vec<(String, i64)>, crate::MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:740` `Ok(vec![])`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:821` `}`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:822` `async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:823` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:824` `_: i64,`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:825` `) -> Result<Vec<(String, i64)>, crate::MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:826` `Ok(vec![])`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1075` ``
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1076` `async fn soft_delete(&self, _: &str) -> Result<(), crate::MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1077` `Ok(())`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1078` `}`
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1079` ``
- `sources/memoria/memoria/crates/memoria-service/src/scheduler.rs:1080` `async fn list_active(&self, _: &str, _: i64) -> Result<Vec<Memory>, crate::MemoriaError> {`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-service/src/service.rs:242`; body evidence.

- `sources/memoria/memoria/crates/memoria-service/src/service.rs:241` `/// Result of a purge operation.`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:242` `pub struct PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:243` `pub purged: usize,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:244` `/// Safety snapshot created before purge. None if snapshot creation failed.`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:245` `pub snapshot_name: Option<String>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:246` `/// Warning message if snapshot creation had issues (quota full, auto-cleanup, etc.)`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:247` `pub warning: Option<String>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:929` `graph.deactivate_by_memory_id(memory_id),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:930` `graph.delete_memory_entity_links(memory_id),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:931` `sql.delete_entity_links_by_memory_id(memory_id),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:932` `);`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:933` `if let Err(e) = r1 {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:934` `tracing::warn!(memory_id, error = %e, "{op}: failed to deactivate graph node");`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:935` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:936` `if let Err(e) = r2 {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:937` `tracing::warn!(memory_id, error = %e, "{op}: failed to delete entity links");`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:938` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:939` `if let Err(e) = r3 {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:940` `tracing::warn!(memory_id, error = %e, "{op}: failed to delete legacy entity links");`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:941` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:942` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:943` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:944` `/// Enqueue a memory for async entity extraction.`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2247` `self.store.insert(&new_mem).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2248` `self.store.soft_delete(memory_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2249` `let mut old_updated = old;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2250` `old_updated.is_active = false;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2251` `old_updated.superseded_by = Some(new_mem.memory_id.clone());`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2252` `self.store.update(&old_updated).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2256` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2257` `pub async fn purge(&self, user_id: &str, memory_id: &str) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2258` `self.purge_on_branch(user_id, None, memory_id).await`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2259` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2260` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2261` `pub async fn purge_on_branch(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2262` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2263` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2264` `branch: Option<&str>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2265` `memory_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2266` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2267` `if self.sql_store.is_some() {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2268` `let sql = self.user_sql_store(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2269` `let (snap, warning) = sql.create_safety_snapshot("purge").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2270` `self.send_edit_log(user_id, "purge", Some(memory_id), None, "", snap.as_deref());`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2271` `let table = sql.table_for_branch(user_id, branch).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2272` `let deactivated = sql.soft_delete_from(&table, memory_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2273` `if deactivated > 0 {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2274` `self.report(StatsEvent::MemoryDeactivated {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2275` `user_id: user_id.to_string(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2276` `});`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2278` `// Graph + entity link cleanup (best-effort, governance fallback covers crash)`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2279` `Self::cleanup_entity_data_for_memory(sql.as_ref(), memory_id, "purge").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2280` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2281` `purged: 1,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2282` `snapshot_name: snap,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2283` `warning,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2284` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2285` `} else {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2286` `self.store.soft_delete(memory_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2287` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2288` `purged: 1,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2289` `snapshot_name: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2290` `warning: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2291` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2292` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2295` `/// Purge multiple memories by IDs with a single audit log entry.`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2296` `pub async fn purge_batch(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2297` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2298` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2299` `ids: &[&str],`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2300` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2301` `self.purge_batch_on_branch(user_id, None, ids).await`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2302` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2303` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2304` `pub async fn purge_batch_on_branch(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2305` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2306` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2307` `branch: Option<&str>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2308` `ids: &[&str],`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2309` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2310` `if self.sql_store.is_some() {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2311` `let sql = self.user_sql_store(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2312` `let (snap, warning) = sql.create_safety_snapshot("purge").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2313` `let table = sql.table_for_branch(user_id, branch).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2314` `for id in ids {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2315` `let deactivated = sql.soft_delete_from(&table, id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2316` `self.send_edit_log(user_id, "purge", Some(id), None, "", snap.as_deref());`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2317` `if deactivated > 0 {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2318` `self.report(StatsEvent::MemoryDeactivated {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2319` `user_id: user_id.to_string(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2320` `});`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2321` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2322` `Self::cleanup_entity_data_for_memory(sql.as_ref(), id, "purge_batch").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2323` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2324` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2325` `purged: ids.len(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2326` `snapshot_name: snap,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2327` `warning,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2328` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2329` `} else {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2330` `for id in ids {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2331` `self.store.soft_delete(id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2332` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2333` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2334` `purged: ids.len(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2335` `snapshot_name: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2336` `warning: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2337` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2338` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2341` `/// Purge memories whose content contains 'topic' (exact text match).`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2342` `pub async fn purge_by_topic(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2343` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2344` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2345` `topic: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2346` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2347` `self.purge_by_topic_on_branch(user_id, None, topic).await`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2348` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2349` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2350` `pub async fn purge_by_topic_on_branch(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2351` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2352` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2353` `branch: Option<&str>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2354` `topic: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2355` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2356` `if self.sql_store.is_some() {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2357` `let sql = self.user_sql_store(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2358` `let (snap, warning) = sql.create_safety_snapshot("purge").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2359` `let table = sql.table_for_branch(user_id, branch).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2360` `let ids = sql.find_ids_by_topic(&table, user_id, topic).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2361` `let reason = format!("topic:{topic}");`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2362` `self.purge_sql_ids(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2363` `user_id,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2364` `sql.as_ref(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2365` `&table,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2366` `&ids,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2370` `.await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2371` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2372` `purged: ids.len(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2373` `snapshot_name: snap,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2374` `warning,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2375` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2376` `} else {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2377` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2378` `purged: 0,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2379` `snapshot_name: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2380` `warning: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2381` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2382` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2384` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2385` `pub async fn purge_by_session_id(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2386` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2387` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2388` `session_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2389` `memory_types: Option<&[MemoryType]>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2390` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2391` `self.purge_by_session_id_on_branch(user_id, None, session_id, memory_types)`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2392` `.await`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2393` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2394` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2395` `pub async fn purge_by_session_id_on_branch(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2396` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2397` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2398` `branch: Option<&str>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2399` `session_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2400` `memory_types: Option<&[MemoryType]>,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2401` `) -> Result<PurgeResult, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2402` `if self.sql_store.is_some() {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2403` `let sql = self.user_sql_store(user_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2404` `let (snap, warning) = sql.create_safety_snapshot("purge").await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2405` `let table = sql.table_for_branch(user_id, branch).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2406` `let ids = sql`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2407` `.find_ids_by_session_id(&table, user_id, session_id, memory_types)`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2408` `.await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2409` `let reason = format!("session:{session_id}");`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2410` `self.purge_sql_ids(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2411` `user_id,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2412` `sql.as_ref(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2413` `&table,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2414` `&ids,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2418` `.await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2419` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2420` `purged: ids.len(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2421` `snapshot_name: snap,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2422` `warning,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2423` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2424` `} else {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2432` `for memory in &memories {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2433` `self.store.soft_delete(&memory.memory_id).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2434` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2435` `Ok(PurgeResult {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2436` `purged: memories.len(),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2437` `snapshot_name: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2438` `warning: None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2439` `})`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2440` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2442` ``
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2443` `async fn purge_sql_ids(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2444` `&self,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2445` `user_id: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2446` `sql: &SqlMemoryStore,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2447` `table: &str,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2452` `if !ids.is_empty() {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2453` `sql.soft_delete_batch_from(table, ids).await?;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2454` `sql.cleanup_entity_data_batch(ids).await;`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2455` `}`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2456` `for id in ids {`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2457` `self.send_edit_log(`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2458` `user_id,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2459` `"purge",`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2460` `Some(id.as_str()),`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2461` `None,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2462` `reason,`
- `sources/memoria/memoria/crates/memoria-service/src/service.rs:2463` `snapshot_name,`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:155`; body evidence.

- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:154` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:155` `self.exec(&format!("DATA BRANCH DELETE TABLE {}", self.branch))`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:156` `.await?;`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:157` `let count: i64 = sqlx::query_scalar(`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:158` `"SELECT COUNT(*) FROM information_schema.tables WHERE table_schema=? AND table_name=?",`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:159` `)`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:165` `if count != 0 {`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:166` `return Err("native delete left a probe table behind".into());`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:167` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:168` `self.branch_created = false;`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:169` `Ok(())`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:170` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:174` `let native = self`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:175` `.exec(&format!("DATA BRANCH DELETE TABLE {}", self.branch))`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:176` `.await;`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:177` `if let Err(error) = native {`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:178` `// Strictly local to this freshly-created probe, never exposed to`
- `sources/memoria/memoria/crates/memoria-storage/src/branch_capability.rs:179` `// user/registry table names. Old builds may have materialized it.`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1037`; body evidence.

- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1036` `/// Deactivate entity links in 'mem_memory_entity_links' for a memory_id.`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1037` `pub async fn delete_memory_entity_links(&self, memory_id: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1038` `let r = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1039` `"DELETE FROM {} WHERE memory_id = ?",`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1040` `self.t("mem_memory_entity_links")`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1041` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1042` `.bind(memory_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1043` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1073` `let sql = format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1074` `"DELETE FROM {} WHERE {conds}",`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1075` `self.t("mem_memory_entity_links")`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1076` `);`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1077` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/graph/store.rs:1078` `for (mid, eid) in chunk {`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:328`; body evidence.

- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:327` `"legacy database still has active snapshots for: {details}. \`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:328` `Delete/recreate those snapshots manually before cutover; this CLI does not materialize \`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:329` `per-user snapshots from legacy shared-db snapshots."`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:330` `)));`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:331` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:332` ``
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:490` `reset_database(&target_url).await?;`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:491` `router.invalidate_user(user_id).await;`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:492` `// Register user in shared DB so runtime can discover it later`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:493` `router.register_user_db(user_id, &target_db).await?;`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:494` `target_db`
- `sources/memoria/memoria/crates/memoria-storage/src/migration.rs:495` `} else {`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-storage/src/router.rs:377`; body evidence.

- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:376` `let _ = sqlx::query(`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:377` `"DELETE FROM mem_user_registry WHERE user_id = ? AND db_name = ?",`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:378` `)`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:379` `.bind(&user_id_owned)`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:380` `.bind(&db_name)`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:381` `.execute(&shared_pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:415` ``
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:416` `pub async fn invalidate_user(&self, user_id: &str) {`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:417` `self.user_db_cache.invalidate(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:418` `self.user_schema_cache.invalidate(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:419` `self.user_store_cache.invalidate(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:420` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:421` ``
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:422` `async fn ensure_user_registry_table(&self) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/router.rs:423` `sqlx::query(`

**operation and route call sites**, body `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2771`; body evidence.

- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2770` `return (Some(name), Some(format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2771` `"⚠️ Snapshot quota was full. Auto-deleted {dropped} oldest safety snapshots to make room. \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2772` `Consider running memory_snapshot_delete(prefix=\"pre_\") to free more space."`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2773` `)));`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2774` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2775` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2776` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2779` `"⚠️ Safety snapshot could not be created (snapshot quota exhausted). \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2780` `Purge proceeded without rollback protection. \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2781` `Run memory_snapshot_delete(prefix=\"pre_\") or memory_snapshot_delete(older_than=\"...\") to free quota."`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2782` `.to_string()`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2783` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2784` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:2785` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3168` `.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3169` `self.active_table_cache.invalidate(state_user.as_ref());`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3170` `Ok(())`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3171` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3172` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3173` `pub async fn invalidate_user_caches(&self, user_id: &str) {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3174` `// Invalidate both the scope key and (if different) the actor key`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3175` `self.active_table_cache.invalidate(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3176` `let state_user = self.state_user(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3177` `if state_user.as_ref() != user_id {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3178` `self.active_table_cache.invalidate(state_user.as_ref());`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3179` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3180` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3181` `// Keep rollback reconciliation scoped to the known per-user governance cooldowns.`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3182` `for operation in [`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3188` `let key = format!("{user_id}:{operation}");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3189` `self.cooldown_cache.invalidate(&key);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3190` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3191` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3192` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3193` `pub async fn register_branch(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3220` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3221` `"UPDATE {branches_table} SET status = 'deleted' WHERE user_id = ? AND name = ?"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3222` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3223` `.bind(user_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3224` `.bind(name)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3225` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3230` `} else {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3231` `self.active_table_cache.invalidate(user_id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3232` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3233` `Ok(())`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3234` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3235` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3439` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3440` `"UPDATE {snapshots_table} SET status = 'deleted' WHERE user_id = ? AND name = ? AND status = 'active'"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3441` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3442` `.bind(user_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3443` `.bind(name)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3444` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3456` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3457` `"UPDATE {snapshots_table} SET status = 'deleted' \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3458` `WHERE user_id = ? AND snapshot_name = ? AND status = 'active'"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3459` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3460` `.bind(user_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3461` `.bind(snapshot_name)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3633` `let res = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3634` `"DELETE FROM {memories_table} \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3635` `WHERE user_id = ? AND is_active = 1 AND trust_tier = ? \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3636` `AND (initial_confidence * EXP(-TIMESTAMPDIFF(DAY, observed_at, NOW()) / {hl})) < {THRESHOLD} \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3637` `LIMIT {BATCH}"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3638` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3665` `let res = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3666` `"DELETE FROM {memories_table} WHERE user_id = ? AND is_active = 0 \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3667` `AND (superseded_by IS NULL OR superseded_by = '') \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3668` `AND updated_at < DATE_SUB(NOW(), INTERVAL 24 HOUR) LIMIT 500"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3669` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3670` `.bind(user_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3696` `let placeholders = ids.iter().map(|_| "?").collect::<Vec<_>>().join(",");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3697` `let sql = format!("DELETE FROM {memories_table} WHERE memory_id IN ({placeholders})");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3698` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3699` `for (id,) in &ids {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3700` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3701` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3713` `let res = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3714` `"DELETE FROM {memories_table} \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3715` `WHERE memory_type = 'tool_result' \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3716` `AND TIMESTAMPDIFF(HOUR, observed_at, NOW()) > ? \`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3717` `LIMIT 5000"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3718` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3733` `/// Returns per-user counts for audit logging.`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3734` `pub async fn archive_stale_working(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3735` `&self,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3736` `stale_hours: i64,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3737` `) -> Result<Vec<(String, i64)>, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3738` `let memories_table = self.t("mem_memories");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3842` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3843` `let mut to_delete: Vec<String> = vec![];`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3844` `let mut deactivated_ids: std::collections::HashSet<String> = Default::default();`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3845` `let mut pairs_checked = 0;`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3846` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3847` `'outer: for group in by_type.values() {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3880` `deactivated_ids.insert(older.clone());`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3881` `to_delete.push(older);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3882` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3883` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3884` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3885` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3886` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3887` `if to_delete.is_empty() {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3888` `return Ok(0);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3889` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3890` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3891` `// Batch DELETE redundant memories (edit_log provides audit trail)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3892` `for chunk in to_delete.chunks(100) {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3893` `let placeholders = chunk.iter().map(|_| "?").collect::<Vec<_>>().join(",");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3894` `let sql = format!("DELETE FROM {memories_table} WHERE memory_id IN ({placeholders})");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3895` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3896` `for id in chunk {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3897` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3898` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3901` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3902` `Ok(to_delete.len() as i64)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3903` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3904` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3905` `/// Rebuild IVF vector index for a table. lists = max(1, rows/50), capped at 1024.`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:3906` `pub async fn rebuild_vector_index(&self, table: &str) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4069` `let _ = sqlx::raw_sql(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4070` `"DATA BRANCH DELETE TABLE {}.{}",`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4071` `quote_ident(&db_name),`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4072` `quote_ident(&table_name)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4073` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4074` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4177` `let sql = format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4178` `"DELETE FROM {stats_table} WHERE memory_id IN ({})",`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4179` `placeholders.join(", ")`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4180` `);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4181` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4182` `for (id,) in &ids {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4201` `let res = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4202` `"DELETE FROM {edit_log_table} WHERE created_at < DATE_SUB(NOW(), INTERVAL ? DAY) LIMIT 1000"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4203` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4204` `.bind(retain_days)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4205` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4206` `.await`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4223` `let res = sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4224` `"DELETE FROM {feedback_table} WHERE created_at < DATE_SUB(NOW(), INTERVAL ? DAY) LIMIT 1000"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4225` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4226` `.bind(retain_days)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4227` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4228` `.await`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4257` `let placeholders = orphans.iter().map(|_| "?").collect::<Vec<_>>().join(",");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4258` `let sql = format!("DELETE FROM {entity_links_table} WHERE id IN ({placeholders})");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4259` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4260` `for (id,) in &orphans {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4261` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4262` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4267` `/// Delete entity links in 'mem_entity_links' for a specific memory_id.`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4268` `pub async fn delete_entity_links_by_memory_id(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4269` `&self,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4270` `memory_id: &str,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4271` `) -> Result<i64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4272` `let table = self.t("mem_entity_links");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4273` `let r = sqlx::query(&format!("DELETE FROM {table} WHERE memory_id = ?"))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4274` `.bind(memory_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4275` `.execute(&self.pool)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4276` `.await`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4277` `.map_err(db_err)?;`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4535` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4536` `"DELETE FROM {lock_table} WHERE lock_key = ? AND holder_id = ?"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4537` `))`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4538` `.bind(key)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4539` `.bind(&self.instance_id)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:4540` `.execute(&mut *conn)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5257` `/// was already inactive or not found , idempotent, not an error).`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5258` `pub async fn soft_delete_from(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5259` `&self,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5260` `table: &str,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5261` `memory_id: &str,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5262` `) -> Result<u64, MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5277` `/// Batch soft-delete: deactivate multiple memories in one round trip per chunk.`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5278` `pub async fn soft_delete_batch_from(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5279` `&self,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5280` `table: &str,`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5281` `ids: &[String],`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5282` `) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5328` `let sql = format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5329` `"DELETE FROM {memory_entity_links_table} WHERE memory_id IN ({placeholders})"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5330` `);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5331` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5332` `for id in chunk {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5333` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5335` `if let Err(e) = q.execute(&mut *conn).await {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5336` `tracing::warn!("batch delete memory_entity_links failed: {e}");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5337` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5338` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5339` `// Delete entity_links`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5340` `let sql =`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5341` `format!("DELETE FROM {entity_links_table} WHERE memory_id IN ({placeholders})");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5342` `let mut q = sqlx::query(&sql);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5343` `for id in chunk {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5344` `q = q.bind(id);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5345` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5346` `if let Err(e) = q.execute(&mut *conn).await {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5347` `tracing::warn!("batch delete entity_links failed: {e}");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5348` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5349` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5350` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:5351` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6846` `store`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6847` `.log_edit("u1", "purge", Some("m2"), None, "test", None)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6848` `.await;`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6849` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6850` `let output = String::from_utf8(logs.lock().unwrap().clone()).expect("utf8 logs");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6851` `assert!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6855` `assert!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6856` `output.contains("operation=purge"),`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6857` `"expected operation field in logs: {output}"`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6858` `);`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6859` `}`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6860` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6894` `store`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6895` `.log_edit("u1", "purge", Some("m1"), None, "closed-test", None)`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6896` `.await;`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6897` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6898` `let output = String::from_utf8(logs.lock().unwrap().clone()).expect("utf8 logs");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:6899` `assert!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7014` ``
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7015` `async fn soft_delete(&self, memory_id: &str) -> Result<(), MemoriaError> {`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7016` `let now = Utc::now().naive_utc();`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7017` `let table = self.t("mem_memories");`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7018` `sqlx::query(&format!(`
- `sources/memoria/memoria/crates/memoria-storage/src/store.rs:7019` `"UPDATE {table} SET is_active = 0, updated_at = ? WHERE memory_id = ?"`

**_invalidate_cache**, body `sources/memoria/plugins/hermes/__init__.py:279`; implementation/call-through.

- `sources/memoria/plugins/hermes/__init__.py:278` `def _invalidate_cache(self):`
- `sources/memoria/plugins/hermes/__init__.py:279` `with self._lock:`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/branches.py:61`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/branches.py:60` `def delete(self, name: str) -> None:`
- `sources/memoria/sdk/python/src/memoria/resources/branches.py:61` `self._client._request("DELETE", f"/v1/branches/{name}")`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/branches.py:148`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/branches.py:147` `async def delete(self, name: str) -> None:`
- `sources/memoria/sdk/python/src/memoria/resources/branches.py:148` `await self._client._arequest("DELETE", f"/v1/branches/{name}")`

**correct**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:494`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:486` `def correct(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:494` `body = _strip_none({"new_content": new_content, "reason": reason, "branch": branch})`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:530`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:523` `def delete(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:531` `self._client._request("DELETE", f"/v1/memories/{id}", params=params)`

**purge**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:543`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:533` `def purge(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:543` `selectors = sum(`

**correct**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:789`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:781` `async def correct(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:789` `body = _strip_none({"new_content": new_content, "reason": reason, "branch": branch})`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:825`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:818` `async def delete(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:826` `await self._client._arequest("DELETE", f"/v1/memories/{id}", params=params)`

**purge**, body `sources/memoria/sdk/python/src/memoria/resources/memories.py:838`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/memories.py:828` `async def purge(`
- `sources/memoria/sdk/python/src/memoria/resources/memories.py:838` `selectors = sum(`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:53`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:40` `def delete(`
- `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:70` `self._client._request("DELETE", f"/v1/snapshots/{name}")`

**delete**, body `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:105`; implementation/call-through.

- `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:97` `async def delete(`
- `sources/memoria/sdk/python/src/memoria/resources/snapshots.py:122` `await self._client._arequest("DELETE", f"/v1/snapshots/{name}")`

</details>

## 3. What reaches the agent

Prefetch caches search rows then assembles with configured context_chars (`sources/memoria/plugins/hermes/__init__.py:351`). Defaults: five hits, 6000 characters; config range is 512..20000 (`sources/memoria/plugins/hermes/config.py:17`; `sources/memoria/plugins/hermes/config.py:64`). Tool writes return bounded receipts; other encoded results exceeding 30000 characters are omitted rather than broken JSON (`sources/memoria/plugins/hermes/__init__.py:634`; `sources/memoria/plugins/hermes/__init__.py:640`). Capture is off by default (`sources/memoria/plugins/hermes/config.py:16`).

Host assembly: `hermes-agent/agent/memory_manager.py:352` joins provider system blocks. `hermes-agent/agent/turn_context.py:1081` requests prefetch for nontrivial turns; `hermes-agent/agent/turn_context.py:120` fences it; `hermes-agent/agent/turn_context.py:136` appends it to API-bound user content. Tool results are a third channel. Optional host spill is at `hermes-agent/agent/memory_manager.py:419`, not a universal provider cap. Built-in memory remains separately controlled (`hermes-agent/agent/system_prompt.py:542`).

<details>
<summary>Context and tool-result assembly quotes</summary>

`_context`, body `sources/memoria/plugins/hermes/__init__.py:113`:

- `sources/memoria/plugins/hermes/__init__.py:116` `# JSON string encoding and escaped delimiters prevent a memory closing the wrapper.`
- `sources/memoria/plugins/hermes/__init__.py:118` `json.dumps(selected + [item], ensure_ascii=False)`
- `sources/memoria/plugins/hermes/__init__.py:122` `if len(candidate) > budget - 200:`
- `sources/memoria/plugins/hermes/__init__.py:124` `selected.append(item)`
- `sources/memoria/plugins/hermes/__init__.py:126` `return "", 0`
- `sources/memoria/plugins/hermes/__init__.py:127` `body = json.dumps(selected, ensure_ascii=False).replace("<", "\\u003c").replace(">", "\\u003e")`
- `sources/memoria/plugins/hermes/__init__.py:128` `return (`

`system_prompt_block`, body `sources/memoria/plugins/hermes/__init__.py:289`:

- `sources/memoria/plugins/hermes/__init__.py:290` `return ""`
- `sources/memoria/plugins/hermes/__init__.py:291` `return (`

`prefetch`, body `sources/memoria/plugins/hermes/__init__.py:324`:

- `sources/memoria/plugins/hermes/__init__.py:332` `return ""`
- `sources/memoria/plugins/hermes/__init__.py:347` `return ""`
- `sources/memoria/plugins/hermes/__init__.py:354` `return ""`
- `sources/memoria/plugins/hermes/__init__.py:358` `return text`
- `sources/memoria/plugins/hermes/__init__.py:361` `return ""`

`queue_prefetch`, body `sources/memoria/plugins/hermes/__init__.py:364`:



`get_tool_schemas`, body `sources/memoria/plugins/hermes/__init__.py:544`:

- `sources/memoria/plugins/hermes/__init__.py:544` `return SCHEMAS if not self._ready or self._local else []`

`handle_tool_call`, body `sources/memoria/plugins/hermes/__init__.py:547`:

- `sources/memoria/plugins/hermes/__init__.py:568` `"limit": args.get("limit", 50),`
- `sources/memoria/plugins/hermes/__init__.py:627` `return _profile_json(result)`
- `sources/memoria/plugins/hermes/__init__.py:629` `return _search_json(result)`
- `sources/memoria/plugins/hermes/__init__.py:631` `# Always return a bounded receipt, below the tested host's`
- `sources/memoria/plugins/hermes/__init__.py:637` `return json.dumps({"success": True, "result": receipt}, ensure_ascii=False)`
- `sources/memoria/plugins/hermes/__init__.py:639` `encoded = json.dumps({"success": True, "result": result}, ensure_ascii=False)`
- `sources/memoria/plugins/hermes/__init__.py:641` `return json.dumps({"success": True, "result_omitted": "response_too_large"})`
- `sources/memoria/plugins/hermes/__init__.py:642` `return encoded`
- `sources/memoria/plugins/hermes/__init__.py:647` `return json.dumps(`
- `sources/memoria/plugins/hermes/__init__.py:656` `return json.dumps({"success": False, "error": str(exc)})`
- `sources/memoria/plugins/hermes/__init__.py:658` `return json.dumps({"success": False, "error": "memoria_internal_error"})`

</details>

## 4. Data access boundaries

MEMORIA_API_KEY is resolved through Hermes scoped secrets, not stored in memoria.json (`sources/memoria/plugins/hermes/config.py:88`; `sources/memoria/plugins/hermes/config.py:26`). Default host api.thememoria.ai; local HTTP allowed only for loopback, remote requires HTTPS (`sources/memoria/plugins/hermes/config.py:7`; `sources/memoria/plugins/hermes/config.py:55`). Direct access `yes` with the same key. Accountless local mode `yes` via self-hosted engine plus locally issued API key: server supports an empty master-key/open mode and generates API keys locally (`sources/memoria/memoria/crates/memoria-api/src/state.rs:162`; `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:38`). Key-required is not the same as requiring a hosted account; local embedding endpoints are configurable (`sources/memoria/memoria/crates/memoria-cli/src/main.rs:116`). Remote telemetry unknown.

Boundary model: same OS user, terminal, mounted filesystem and allowed network. Direct reachability means another client can use the same file/endpoint and available credentials, not unauthenticated access or encryption bypass. Hermes setup persists declared secrets through `hermes-agent/hermes_cli/memory_setup.py:321` and `hermes-agent/hermes_cli/config.py:2697`. These are possible configured paths; no live credentials were read.

Host credential-file detail: `get_hermes_home()/.env` is resolved at `hermes-agent/hermes_cli/config.py:473`. New files are secured to 0600 at `hermes-agent/hermes_cli/config.py:506`; the writer preserves an existing mode at `hermes-agent/hermes_cli/config.py:2544` and `:2562`. Therefore 0600 is not an unconditional mode claim for every existing profile.

<details>
<summary>Paths, environment, permissions, egress, telemetry and process-launch evidence</summary>

- `sources/memoria/memoria/crates/memoria-service/src/config.rs:132` `.unwrap_or_else(|_| "https://api.openai.com/v1".to_string()),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:324` `url: "https://api.example.com/v1".into(),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:341` `r#"[{"url":"https://a.com/v1","api_key":"key-a"},{"url":"https://b.com/v1","api_key":"key-b"}]"#,`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:351` `assert_eq!(eps[0].url, "https://a.com/v1");`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:353` `assert_eq!(eps[1].url, "https://b.com/v1");`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:366` `Some(r#"[{"url":"https://multi.com/v1","api_key":"mk"}]"#),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:368` `("EMBEDDING_BASE_URL", Some("https://single.com/v1")),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:376` `assert_eq!(eps[0].url, "https://multi.com/v1");`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:387` `("EMBEDDING_BASE_URL", Some("https://single.com/v1")),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:394` `assert_eq!(eps[0].url, "https://single.com/v1");`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:423` `Some(r#"[{"url":"https://a.com/v1","api_key":"k"}]"#),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:440` `("EMBEDDING_BASE_URL", Some("https://single.com/v1")),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:456` `Some(r#"[{"url":"https://a.com/v1","api_key":"k"}]"#),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:458` `("EMBEDDING_BASE_URL", Some("https://single.com/v1")),`
- `sources/memoria/memoria/crates/memoria-service/src/config.rs:475` `Some(r#"[{"url":"https://a.com/v1","api_key":"k"}]"#),`

- `sources/memoria/plugins/hermes/__init__.py:676` `"description": "Memoria API Key (log in at https://thememoria.ai)",`
- `sources/memoria/plugins/hermes/__init__.py:712` `with os.fdopen(os.open(backup, os.O_CREAT | os.O_EXCL | os.O_WRONLY, 0o600), "wb") as f:`

- `sources/memoria/plugins/hermes/config.py:7` `DEFAULT_URL = "https://api.thememoria.ai"`
- `sources/memoria/plugins/hermes/config.py:88` `return (get_secret("MEMORIA_API_KEY") or "").strip()`

- `sources/memoria/plugins/hermes/config_schema.py:13` `docs_url="https://thememoria.ai",`
- `sources/memoria/plugins/hermes/config_schema.py:22` `ProviderField(key="api_url", label="API origin", default="https://api.thememoria.ai"),`

- `sources/memoria/plugins/hermes/outbox.py:20` `directory.mkdir(parents=True, exist_ok=True, mode=0o700)`
- `sources/memoria/plugins/hermes/outbox.py:21` `directory.chmod(0o700)`
- `sources/memoria/plugins/hermes/outbox.py:25` `self.path.touch(mode=0o600, exist_ok=True)`
- `sources/memoria/plugins/hermes/outbox.py:26` `self.path.chmod(0o600)`

</details>

## 5. Reproducibility

No engine package pin in the provider: HTTP server revision `unpinned`; retained Rust/Python engine versions are not a deployment pin. Provider packaging dependencies are listed below.

Retained lock/checksum files: `sources/memoria/plugins/openclaw/package-lock.json:1`. Upstream completeness is unknown: fetch.py:15 excludes .lock files, fetch.py:12 excludes build/vendor directories. Absence here is not absence upstream.

- `sources/memoria/plugins/hermes/plugin.yaml:2` `version: 0.1.1`
- `sources/memoria/plugins/hermes/plugin.yaml:4` `requires_hermes: ">=0.21.5"`

- `sources/memoria/plugins/hermes/pyproject.toml:3` `version = "0.1.1"`
- `sources/memoria/plugins/hermes/pyproject.toml:5` `requires-python = ">=3.11"`
- `sources/memoria/plugins/hermes/pyproject.toml:6` `dependencies = ["httpx>=0.27,<1"]`
- `sources/memoria/plugins/hermes/pyproject.toml:14` `target-version = "py311"`

## 6. Tests

Adapter tests live under plugins/hermes/tests; forget/scope/cache cases and Rust purge tests are distinct and counted below. No runtime execution. Adapter tool-cycle test asserts the forget operation; cache/scope tests are separate. (`sources/memoria/plugins/hermes/tests/test_provider.py:19`).

Adapter-related declarations: **62**. Python AST test_ functions, fixtures excluded; literal TS/JS it/test, Go Test*, Rust test_* declarations. No parametrization expansion, collection, execution, or coverage measurement. Other rows are engine test candidates selected by mutation names; names alone do not verify forget coverage.

| file | declarations | adapter-related | forget-related declarations |
|---|---:|---|---|
| `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:1` | 35 | no | `test_delete_single_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:325`; `test_purge_bulk_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:373`; `test_purge_by_topic_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:429`; `test_purge_by_topic_special_chars_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:472`; `test_purge_by_topic_batch_perf_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:511`; `test_purge_by_session_id_with_memory_types_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:544`; `test_admin_delete_user_verify_db` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:985`; `test_delete_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:2276`; `test_purge_bulk_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:2323`; `test_purge_topic_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_db_verify.rs:2367` |
| `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:1` | 190 | no | `test_api_list_excludes_deleted` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:1188`; `test_api_delete` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:1579`; `test_api_purge_bulk` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:1608`; `test_api_key_cannot_delete_other_users_memory` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:2914`; `test_api_delete_nonexistent` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:3480`; `test_remote_correct_purge` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:3756`; `test_remote_purge_by_topic` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:4520`; `test_remote_purge_by_session_id` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:4545`; `test_remote_purge_rejects_invalid_memory_types_locally` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:4668`; `test_api_key_auth_uses_batcher_not_fire_and_forget` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:9858`; `test_remote_purge_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:10125`; `test_remote_purge_batch_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:10174`; `test_remote_purge_topic_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-api/tests/api_e2e.rs:10206` |
| `sources/memoria/memoria/crates/memoria-api/tests/api_multi_db_purge.rs:1` | 2 | no | `test_multidb_purge_paths_use_qualified_tables` `sources/memoria/memoria/crates/memoria-api/tests/api_multi_db_purge.rs:127` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:1` | 50 | no | `test_cannot_delete_main` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:1475`; `test_delete_nonexistent_branch` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:1492`; `test_active_branch_resets_on_delete` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:1657`; `test_purge_on_branch_isolated_from_main` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:2183`; `test_purge_by_topic_on_branch_isolated_from_main` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:2232`; `test_purge_batch_on_branch_isolated_from_main` `sources/memoria/memoria/crates/memoria-mcp/tests/branch_e2e.rs:2286` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:1` | 52 | no | `test_purge_single` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:554`; `test_purge_batch` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:573`; `test_purge_topic` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:602`; `test_purge_session_id_with_memory_types` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:642`; `test_purge_no_target` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:699`; `test_purge_short_topic` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:713`; `test_purge_topic_then_search_empty` `sources/memoria/memoria/crates/memoria-mcp/tests/core_tools_e2e.rs:1573` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:1` | 19 | no | `test_purge_single_all_fields` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:325`; `test_purge_batch_all_fields` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:375`; `test_purge_topic_all_fields` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:429`; `test_purge_rollback_restores_memory` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:617`; `test_api_purge_edit_log_all_fields` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:928`; `test_api_purge_topic_edit_log_all_fields` `sources/memoria/memoria/crates/memoria-mcp/tests/edit_log_e2e.rs:995` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/graph_e2e.rs:1` | 38 | no | `test_purge_deactivates_graph_node` `sources/memoria/memoria/crates/memoria-mcp/tests/graph_e2e.rs:626`; `test_purge_cleans_entity_links` `sources/memoria/memoria/crates/memoria-mcp/tests/graph_e2e.rs:1182`; `test_purge_batch_cleans_graph_and_entity_links` `sources/memoria/memoria/crates/memoria-mcp/tests/graph_e2e.rs:1268` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/mcp_e2e.rs:1` | 6 | no | `test_e2e_correct_purge` `sources/memoria/memoria/crates/memoria-mcp/tests/mcp_e2e.rs:86` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/perf_optimizations_e2e.rs:1` | 12 | no | `test_active_table_cache_hit_and_invalidation` `sources/memoria/memoria/crates/memoria-mcp/tests/perf_optimizations_e2e.rs:58` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:1` | 22 | no | `test_snapshot_delete_by_prefix` `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:346`; `test_snapshot_delete_batch_names` `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:407`; `test_snapshot_delete_is_scoped_to_owner` `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:854`; `test_snapshot_delete_older_than` `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:904`; `test_delete_releases_quota` `sources/memoria/memoria/crates/memoria-mcp/tests/snapshot_e2e.rs:1246` |
| `sources/memoria/memoria/crates/memoria-mcp/tests/tools_unit.rs:1` | 9 | no | `test_tool_memory_purge` `sources/memoria/memoria/crates/memoria-mcp/tests/tools_unit.rs:277` |
| `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:1` | 18 | no | `test_observe_exclusions_retain_inactive_facts_after_correct_or_purge` `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:260`; `test_purge` `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:476`; `test_list_active_excludes_deleted` `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:499`; `test_purge_by_session_id_filters_memory_type` `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:537`; `test_purge_by_session_id_fallback_is_not_capped` `sources/memoria/memoria/crates/memoria-service/tests/service_unit.rs:592` |
| `sources/memoria/memoria/crates/memoria-storage/tests/branch_ops.rs:1` | 11 | no | `test_active_table_fallback_when_branch_deleted` `sources/memoria/memoria/crates/memoria-storage/tests/branch_ops.rs:171`; `test_active_table_fallback_when_branch_deleted_qualifies_main_for_routed_store` `sources/memoria/memoria/crates/memoria-storage/tests/branch_ops.rs:199` |
| `sources/memoria/memoria/crates/memoria-storage/tests/mo_compat.rs:1` | 7 | no | `test_07_update_soft_delete` `sources/memoria/memoria/crates/memoria-storage/tests/mo_compat.rs:186` |
| `sources/memoria/memoria/crates/memoria-storage/tests/store_crud.rs:1` | 30 | no | `test_soft_delete` `sources/memoria/memoria/crates/memoria-storage/tests/store_crud.rs:107` |
| `sources/memoria/memoria/crates/memoria-storage/tests/vector_index_ops.rs:1` | 18 | no | `test_cleanup_stale_deletes_broken_chain` `sources/memoria/memoria/crates/memoria-storage/tests/vector_index_ops.rs:740` |
| `sources/memoria/plugins/hermes/tests/cloud_smoke.py:1` | 0 | yes | none |
| `sources/memoria/plugins/hermes/tests/conftest.py:1` | 0 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_capture_dedup.py:1` | 13 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_client_config.py:1` | 6 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_cloud_smoke.py:1` | 1 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_diagnostics.py:1` | 1 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_provider.py:1` | 17 | yes | none |
| `sources/memoria/plugins/hermes/tests/test_reliability.py:1` | 24 | yes | `test_late_search_cannot_repopulate_invalidated_cache` `sources/memoria/plugins/hermes/tests/test_reliability.py:209` |
| `sources/memoria/plugins/openclaw/openclaw/__tests__/http-client.test.ts:1` | 27 | no | `3.41 memory_purge → ` `sources/memoria/plugins/openclaw/openclaw/__tests__/http-client.test.ts:231` |
| `sources/memoria/plugins/openclaw/openclaw/__tests__/parsers.test.ts:1` | 16 | no | `4.9 purge → ` `sources/memoria/plugins/openclaw/openclaw/__tests__/parsers.test.ts:130`; `4.10 purge with zero → ` `sources/memoria/plugins/openclaw/openclaw/__tests__/parsers.test.ts:137` |
| `sources/memoria/sdk/python/tests/integration/test_e2e.py:1` | 12 | no | `test_store_retrieve_delete` `sources/memoria/sdk/python/tests/integration/test_e2e.py:128`; `test_purge_by_session` `sources/memoria/sdk/python/tests/integration/test_e2e.py:152`; `test_branch_create_list_checkout_delete` `sources/memoria/sdk/python/tests/integration/test_e2e.py:205` |
| `sources/memoria/sdk/python/tests/unit/test_branches.py:1` | 9 | no | `test_delete_branch` `sources/memoria/sdk/python/tests/unit/test_branches.py:58` |
| `sources/memoria/sdk/python/tests/unit/test_memories.py:1` | 50 | no | `test_delete_happy_path` `sources/memoria/sdk/python/tests/unit/test_memories.py:483`; `test_purge_by_ids` `sources/memoria/sdk/python/tests/unit/test_memories.py:493`; `test_purge_by_topic` `sources/memoria/sdk/python/tests/unit/test_memories.py:503`; `test_purge_by_session_with_types` `sources/memoria/sdk/python/tests/unit/test_memories.py:509`; `test_purge_no_selector_raises` `sources/memoria/sdk/python/tests/unit/test_memories.py:517`; `test_purge_multiple_selectors_raises` `sources/memoria/sdk/python/tests/unit/test_memories.py:522`; `test_purge_memory_types_without_session_id_raises` `sources/memoria/sdk/python/tests/unit/test_memories.py:527` |
| `sources/memoria/sdk/python/tests/unit/test_memories_async.py:1` | 13 | no | `test_purge_by_ids` `sources/memoria/sdk/python/tests/unit/test_memories_async.py:132` |
| `sources/memoria/sdk/python/tests/unit/test_snapshots.py:1` | 9 | no | `test_delete_single` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:37`; `test_delete_bulk_names` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:42`; `test_delete_prefix` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:47`; `test_delete_older_than` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:52`; `test_delete_no_selector_raises` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:57`; `test_delete_multiple_selectors_raises` `sources/memoria/sdk/python/tests/unit/test_snapshots.py:62` |

## 7. Corrections to B2

Old values below are copied from the triage row (TRIAGE.md), not reconstructed from memory. A refinement narrows an over-broad field; it need not reverse its original value. Counts are distinct field paths, not the number of factual errors.

- `engine_delete.e`: `["sdk/python/src/memoria/resources/memories.py:533"]` → SDK purge is not the engine body. Rust DELETE dispatches purge_on_branch; adapter forget is one subject-scoped ID and invalidates its cache. Evidence: `sources/memoria/plugins/hermes/__init__.py:609`, `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:613`.
- `account.v`: `"required service key"` → Required service key does not imply hosted signup: self-hosted server generates local keys and supports open admin mode. Evidence: `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:38`, `sources/memoria/memoria/crates/memoria-api/src/state.rs:162`.
- `engine_delete.scope`: `"Named deletion/retraction endpoint or method, not a guarantee of physical erasure. Its reach may overlap an agent tool; lack of a native tool is counted separately."` → `yes`: memoria_forget validates the ID, GETs it in the configured branch, checks subject_id, then DELETE `/v1/memories/{id}` (`sources/memoria/plugins/hermes/__init__.py:598`; `sources/memoria/plugins/hermes/__init__.py:609`). memoria_update uses PUT `/v1/memories/{id}/correct`, a separate correction operation (`sources/memoria/plugins/hermes/__init__.py:603`). Writes invalidate adapter cache (`sources/memoria/plugins/hermes/__init__.py:625`). Rust delete calls purge_on_branch (`sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:613`). Purge supports IDs, topic and session selectors (`sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:628`). Derived reach `partial`: branch-local purge and graph/SQL cleanup are not a promise to remove independent snapshots, branches, exported copies or host history. Evidence: `sources/memoria/plugins/hermes/__init__.py:598`, `sources/memoria/plugins/hermes/__init__.py:609`, `sources/memoria/plugins/hermes/__init__.py:603`, `sources/memoria/plugins/hermes/__init__.py:625`, `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:613`, `sources/memoria/memoria/crates/memoria-api/src/routes/memory.rs:628`.
- `context.mechanism.v`: `"Source expressions below identify retrieval, cached/static returns, lifecycle capture, and limits. A hook definition is not a guarantee it emits nonempty context each turn; branches and configuration apply."` → Prefetch caches search rows then assembles with configured context_chars (`sources/memoria/plugins/hermes/__init__.py:351`). Defaults: five hits, 6000 characters; config range is 512..20000 (`sources/memoria/plugins/hermes/config.py:17`; `sources/memoria/plugins/hermes/config.py:64`). Tool writes return bounded receipts; other encoded results exceeding 30000 characters are omitted rather than broken JSON (`sources/memoria/plugins/hermes/__init__.py:634`; `sources/memoria/plugins/hermes/__init__.py:640`). Capture is off by default (`sources/memoria/plugins/hermes/config.py:16`). Evidence: `sources/memoria/plugins/hermes/__init__.py:351`, `sources/memoria/plugins/hermes/config.py:17`, `sources/memoria/plugins/hermes/config.py:64`, `sources/memoria/plugins/hermes/__init__.py:634`, `sources/memoria/plugins/hermes/__init__.py:640`, `sources/memoria/plugins/hermes/config.py:16`.
- `sec.telemetry.v`: `"unknown: no emitter identified in the selected adapter files; external dependencies not cleared"` → MEMORIA_API_KEY is resolved through Hermes scoped secrets, not stored in memoria.json (`sources/memoria/plugins/hermes/config.py:88`; `sources/memoria/plugins/hermes/config.py:26`). Default host api.thememoria.ai; local HTTP allowed only for loopback, remote requires HTTPS (`sources/memoria/plugins/hermes/config.py:7`; `sources/memoria/plugins/hermes/config.py:55`). Direct access `yes` with the same key. Accountless local mode `yes` via self-hosted engine plus locally issued API key: server supports an empty master-key/open mode and generates API keys locally (`sources/memoria/memoria/crates/memoria-api/src/state.rs:162`; `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:38`). Key-required is not the same as requiring a hosted account; local embedding endpoints are configurable (`sources/memoria/memoria/crates/memoria-cli/src/main.rs:116`). Remote telemetry unknown. Evidence: `sources/memoria/plugins/hermes/config.py:88`, `sources/memoria/plugins/hermes/config.py:26`, `sources/memoria/plugins/hermes/config.py:7`, `sources/memoria/plugins/hermes/config.py:55`, `sources/memoria/memoria/crates/memoria-api/src/state.rs:162`, `sources/memoria/memoria/crates/memoria-api/src/routes/auth.rs:38`, `sources/memoria/memoria/crates/memoria-cli/src/main.rs:116`.
- `repro.decls`: `["plugins/hermes/pyproject.toml:3", "plugins/hermes/pyproject.toml:5", "plugins/hermes/pyproject.toml:6", "plugins/hermes/pyproject.toml:14"]` → No engine package pin in the provider: HTTP server revision `unpinned`; retained Rust/Python engine versions are not a deployment pin. Provider packaging dependencies are listed below. Evidence: `sources/memoria/plugins/openclaw/package-lock.json:1`, `sources/memoria/plugins/hermes/plugin.yaml:2`, `sources/memoria/plugins/hermes/plugin.yaml:4`, `sources/memoria/plugins/hermes/pyproject.toml:3`.
- `tests.count`: `1217` → 62 adapter-related declarations using the stated parser/scope; B2 counted a different, repository-wide inventory. This is a scope refinement, not a count of executed or passing tests. Adapter tests live under plugins/hermes/tests; forget/scope/cache cases and Rust purge tests are distinct and counted below. No runtime execution. Adapter tool-cycle test asserts the forget operation; cache/scope tests are separate. (`sources/memoria/plugins/hermes/tests/test_provider.py:19`). Evidence: `sources/memoria/plugins/hermes/tests/test_provider.py:19`.

MATRIX counts these 7 explicit field corrections/refinements. Sections 1-6 provide the body and test inventories. Test-name candidates are not passing tests or measured coverage.
