![mayfly, issue 0: memory plugins. a field of boxes, five of them carry the marks of the plugins in the pilot](assets/mayfly-header.png)

# mayfly

mayfly gives one Hermes Agent plugin a fresh box, runs up to five scripted sessions, keeps the session record and removes the box.

## issue 0 · memory plugins, a pilot

five memory plugins, one run each, 16 Nov 2026. the agent is told eight made-up facts, asked about them, told one of them changed, then asked again.

| plugin | recall | correction (Porto to Lisbon) | forget tool in the plugin |
|---|---|---|---|
| Mem0 | 9/9 | pass | yes |
| Hindsight | 9/9 | partial · the agent deleted rows in Postgres with SQL | no |
| OpenViking | 9/9 | pass | yes |
| AgentMemory | 9/9 | partial · the agent called the engine's forget directly | no |
| Honcho | not run · our runner broke | not run | yes |

![the same table as a card: recall, correction and forget tool for Mem0, Hindsight, OpenViking, AgentMemory and Honcho](assets/mayfly-outcomes-16x9.png)

recall alone does not separate them: Hermes Agent's built-in memory also scored 9/9 in our control runs (their exports are not in this repo). and with built-in memory switched off and no plugin, the agent switched it back on by itself.

- [issue-0/FACTS.md](issue-0/FACTS.md) · results, one line each, with the session it comes from
- [issue-0/METHOD.md](issue-0/METHOD.md) · setup, the five prompts, the model, what was not done
- [issue-0/sessions/](issue-0/sessions/) · masked session exports of runs 34, 36, 39, 46, 47 and 48
- [issue-0/boxes.json](issue-0/boxes.json) · all 36 boxes of the episode, created and removed
- [issue-0/outcomes.csv](issue-0/outcomes.csv) · the table as data
- [issue-0/forget-scan/](issue-0/forget-scan/) · static scan of 130 memory plugins for a forget path

## tenure #7 · approval prompt redaction

tenure asks for approval before risky tool calls. its prompt used to show the key from the command. 1.1.0 before, 1.1.2 after, 19 Nov 2026: 28 of 28 offline cases pass, and live in Hermes Agent the key is hidden in all 3 fixture commands. code like `<(printf ...)` stays visible by design.

- [tenure-7/RECEIPT.md](tenure-7/RECEIPT.md) · before and after, live and offline, word for word

the runner is not published.

## license

text and data in this repository: [CC BY 4.0](LICENSE). plugin source quoted inside session files stays under its own license. plugin logos in the images under [assets/](assets/) belong to their owners.
