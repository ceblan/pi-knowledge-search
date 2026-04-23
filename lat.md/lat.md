This directory defines the high-level concepts, business logic, and architecture of this project using markdown. It is managed by [lat.md](https://www.npmjs.com/package/lat.md) — a tool that anchors source code to these definitions. Install the `lat` command with `npm i -g lat.md` and run `lat --help`.

- [[architecture]] — Overall extension architecture, component graph, session lifecycle, and data flow.
- [[chunking]] — Markdown-aware content chunking strategy with heading, paragraph, and hard splitting.
- [[configuration]] — Config file format, environment variable overrides, provider types, and defaults.
- [[embedding]] — Embedding interface, provider implementations, rate limiting, and batch processing.
- [[extension-interface]] — Pi lifecycle hooks, slash commands, and the knowledge_search tool.
- [[index-store]] — Persistent JSON vector index with sync, search, and file operations.
- [[kb-searcher]] — Bedrock Knowledge Base search integration with result merging.
- [[sync-worker]] — Background child process for indexing that reports results via stdout.
- [[tests]] — Unit test specifications for config, chunker, and index-store modules.
- [[last-commit]] — Git commit tracker for documentation coverage.
