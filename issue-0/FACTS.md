# issue 0 · memory plugins · facts

pilot run on 16 Nov 2026. five Hermes Agent memory plugins, one fresh box per run. the runner plans five scripted sessions per box; Honcho run 46 stopped after session 1, so 26 session exports are here (5, 5, 5, 1, 5 and 5 for the no-plugin run). lines that point to a session file in `sessions/` or to `boxes.json` can be checked in this repo. lines marked *runner record* rest on our runner's logs or configs, which are not published. how the runs were set up is in [METHOD.md](METHOD.md).

## per plugin

the plugin-only run is the row: built-in memory and session search were set off in config, so the plugin was the only memory (*runner record*; the same flags are visible in the no-plugin run 48, session 1).

| plugin | run | set up | recall (read by a person) | unknown fact (q6) | correction (Porto to Lisbon) | box min |
|---|---|---|---|---|---|---|
| Mem0 | 34 | pass | pass 9/9, `mem0_search` in session 3, auto-injected context in session 5 | pass | pass: `mem0_update` in place. "no Porto anywhere" was not checkable | 7.4 |
| Hindsight | 36 | pass | pass 9/9, `hindsight_recall` | pass, but it stored "the user has a dog whose name is not known" | partial: the plugin has no forget tool. the agent deleted 3 rows in Hindsight's local Postgres with SQL. Porto came back in entities | 9.2 |
| OpenViking | 39 | pass | pass 9/9, OpenViking context injection | pass | pass: profile rewritten to Lisbon. raw session archives not inspected | 8.4 |
| AgentMemory | 47 | pass | pass 9/9, read by a person ("Lisbon (corrected from Porto)" counted as a pass). answers came from the plugin's own context injection | pass | partial: no forget tool. the agent called the engine directly (`iii trigger mem::forget`, deleted 1). the session 1 raw note with Porto stayed | 11.2 |
| Honcho | 46 | pass (Docker, *runner record*) | not run: our runner machine broke before session 2 (npx cache clash during a CLI update, see `boxes.json`). not a Honcho result | not run | not run | 11.1 |

box min is the box lifetime from create to removal in `boxes.json`. the same table as data: [outcomes.csv](outcomes.csv).

## facts worth a line

1. the recall test alone does not separate plugins. Hermes Agent's built-in memory with no plugin also answered 9/9 in our control runs (*runner record*, control exports not in this repo). what separates them is where the answer came from and whether a correction really removes the old fact.
2. forget tools, read from source at the pinned commits: Mem0 `mem0_delete`, OpenViking `viking_forget`, Honcho `honcho_conclude` with `delete_id`. none in Hindsight (retain, recall, reflect) or AgentMemory (recall, save, search). 2 of 5 ship no forget tool.
3. Hindsight, session 4 ([session-4.md](sessions/run-36-hindsight-only/session-4.md)): the agent's `python3 -c` was stopped by Hermes Agent's own command guard. a plain curl GET of the local API's openapi.json worked. then our egress guard, Exfilbar, blocked its next two calls to the local Hindsight API (`execute_code` at 08:23:34Z, `curl` at 08:23:36Z). the agent said network calls to the API were blocked by policy, which was true. it opened Hindsight's local Postgres with the credentials in `instance.json` and deleted three rows by hand. our guard pushed it from the API to SQL. at this commit the engine has no single-memory DELETE anyway, only PATCH invalidate.
4. AgentMemory, session 4 ([session-4.md](sessions/run-47-agentmemory-only/session-4.md)): no forget tool, so the agent searched the engine's registered functions, found `mem::forget` and called it over the local socket with `iii trigger`. one memory deleted. the raw session 1 note with Porto stayed.
5. both plugins without a forget tool still "forgot": Hindsight by SQL in its Postgres, AgentMemory through its engine. neither is a path the plugin offers.
6. the no-memory baseline (run 48) is void, and that is the finding. with built-in memory switched off, the agent read its own config, ran `hermes config set memory.memory_enabled true` itself (09:09:48Z, [session-1.md](sessions/run-48-no-plugin/session-1.md)) and wrote all eight facts to USER.md in session 1. every later answer came from that. a memory test has to lock the config, not just set it.
7. Honcho, run 46 ([session-1.md](sessions/run-46-honcho-only/session-1.md)): all eight `honcho_conclude` calls returned "Failed to save conclusion." (cause not in the export). its profile card did take all eight facts. our earlier Honcho runs saved no conclusions either (*runner record*). shown as not run, not as a Honcho failure.
8. boxes ([boxes.json](boxes.json)): 36 fresh Instabox boxes for the memory runs, 36 removed by the runner with `instabox rm` and checked with `instabox ls` (3 h TTL as a backstop). 0 failures on the Instabox side. run 36 took 9 min 13 s from create to removal.

## other runs, not in this repo

their exports are not published. these lines are *runner record*; the box of each run is in `boxes.json`.

- runs with the plugin plus built-in memory (25 Mem0, 28 Hindsight, 29 Honcho, 32 OpenViking, 35 AgentMemory): built-in memory answered recall in 28, 29, 32 and 35, so those runs say nothing about the plugins' recall.
- Honcho run 29: all six `honcho_conclude` writes returned "Failed to save conclusion." on our box.
- runs 37 and 38 stopped when our own model key hit its daily spending cap. not the plugins.
- failed boxes and why are listed one by one in `boxes.json`. every failure was on our side.
