# forget scan · 130 memory plugins

a static read of the source of Hermes Agent memory plugins: 130 listed, 129 read, each at a pinned commit. no plugin was installed, imported or run for this scan (our record; the scan's working logs are not in this repo).

- [MANIFEST.json](MANIFEST.json): name, repository and commit for each plugin. `scope-recall` returned 404 and was not read.
- [TRIAGE.md](TRIAGE.md): all 130. kind of integration, whether the plugin exposes a forget tool, whether the engine behind it can delete, where it stores data, whether it needs an account. every claim links to the line it rests on.
- [MATRIX.md](MATRIX.md) and [DEEP/](DEEP/): 21 plugins read in depth. forget granularity, derived copies, whether the store is reachable without the plugin, where credentials sit on disk.

`forget_tool = yes` means the plugin exposes an operation that deletes, retracts, archives or retires stored content. it does not mean the bytes are gone from disk.

how the scan was made is our record, not checkable from this folder: the scan was written with a code model. a person checked the forget tool column against source for the five pilot plugins (Mem0, Hindsight, OpenViking, AgentMemory, Honcho). the other rows have not yet been checked line by line. if a row is wrong, open an issue with the line.

links point to the plugin repositories at the pinned commits. `hermes-agent/<path>` means NousResearch/hermes-agent at `bcf7dffeddded34ec1285a8d30aecf8e83ca502e`. `sources/<name>/<path>` in the text means the plugin's repository from the manifest, at its commit.

quoted source is shown as published, except: telemetry ingest keys are shown as `phc_[key cut]` and non-loopback example IP addresses as `[ip]`.
