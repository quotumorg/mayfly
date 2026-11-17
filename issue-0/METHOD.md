# issue 0 · method

the runner code is not published. this file describes what it did. what the session exports and `boxes.json` show can be checked here; the rest of this file is our runner's record (configs and logs), which is not published.

## the box

- one fresh Instabox box per run, from one snapshot with Hermes Agent v0.23.1. the box is removed at the end of the run with `instabox rm` and checked with `instabox ls`. every box also had a 3 h TTL as a backstop.
- one plugin per box, installed from its repository at a pinned commit.
- Exfilbar, our egress guard plugin for Hermes Agent, was on in every box (runner config; visible in the exports of runs 36, 47 and 48). it blocks network transfers it cannot resolve to an allowed destination. in run 36 it blocked the agent's calls to Hindsight's local API (see [FACTS.md](FACTS.md), fact 3).
- plugin-only runs: Hermes Agent's built-in memory and session search were switched off in config, so the plugin was the only memory (runner config; the flags are visible in run 48, session 1). the config was set, not locked. in the no-plugin baseline the agent switched built-in memory back on by itself (fact 6), so that run is void.

## the five sessions

each session is a separate one-shot Hermes Agent session in the same box. the user turns, word for word:

1. "Please remember these facts about me for future conversations: I am allergic to peanuts. My cat is named Pickle. My timezone is Europe/Lisbon. I use metric units. I am restoring a lighthouse. The deadline is 3 March 2027. The budget is 4,200 euros. I currently live in Porto."
2. "Explain why a glass of cold water gets droplets on the outside, in three short sentences."
3. "Recall what I previously told you. Answer each question on its own line labelled q1: through q6:. If you do not know, say so. q1: What is my cat called? q2: What timezone do I use? q3: What is my lighthouse restoration budget? q4: Today is 24 February 2027. How many days remain until my deadline? q5: Suggest a snack that is safe for my allergy and state what you must avoid. q6: What is my dog called?"
4. "I have moved to Lisbon. Porto is incorrect as my current city. Correct your memory, show what you remember about my city, and delete the incorrect Porto fact from memory."
5. "Answer each question on its own line labelled q1: through q3:, using what you remember. q1: What is my current city? q2: What is my cat called? q3: What units do I prefer?"

all facts are made up. q6 asks for a dog that was never mentioned: the right answer is "I don't know". recall is 9 answers (q1-q5 of session 3, q1-q3 of session 5, plus q6). a person read every answer and checked where it came from: a plugin tool call, injected plugin context, or something else.

## the model

the specs asked for deepseek-v4-1-flash. every session export records kimi-k3: the snapshot's entrypoint rewrote the model before every command, and we found it after the runs. the agent model was kimi-k3 in every export here, so the comparison between plugins holds, but this is not a deepseek result. plugins that call a model of their own keep their own setting: Hindsight's config in run 36 names `deepseek-v4-1-flash` for its internal calls (session 4). the per-run price estimates in our run cards used the wrong model price and are not quoted here.

by setup, the model key never entered the box (where the exports show the key setting, they show `[proxy-placeholder]`). the box got only a placeholder from Instabox's secret proxy (`instabox create --secrets <name>`), and the proxy swapped in the real key on the way out. returned placeholders were redacted before parsing.

## cost

per-run cost was not measured. whole-key spend for one batch of 4 runs (46-49): $1.64, about $0.41 a run. Hindsight's own counters for run 36: 97k LLM tokens and 37 embedding calls inside the plugin. both are from provider and runner records, not in this repo.

## what the session files are

`sessions/run-<n>-<plugin>/session-<k>.md` is the Hermes Agent session export of that run, rendered to markdown: user turns, the model's visible thinking, tool calls and tool results. the system prompt is left out. our runner's own commands are not included. tool results longer than 3000 characters are cut, and the cut is marked.

masking: identifiers, local paths, IP addresses, session ids and credential values are replaced by `•••` plus 12 hex characters. the same value always gets the same tag, so you can follow it across a file. a shortened id in the agent's text gets the tag of the full id. hex dumps of store files are cut whole, since they spell ids byte by byte. database passwords and credential pairs are shown as `[masked]` and the provider endpoint as `[provider url]`. long strings our scanner could not classify are replaced by `[cut: long string]`. a few common words the scanner had taken for private values were kept as words after a person checked them.

## what this pilot did not do

- Honcho plugin-only did not finish: our runner machine broke before session 2.
- runs 37 and 38 hit our model key's daily cap.
- one run per plugin, no repeats, so no spread.
- no check that a "deleted" fact is gone from disk. we read what the plugin and the agent showed, not the stores.
- built-in memory config was set off, not locked, and prompt files were writable.
- no terminal recording of the sessions. the session exports are the record.
- the other 125 Hermes Agent memory plugins were not run. they are in the static scan in [forget-scan/](forget-scan/).
