# Architecture

The pi-knowledge-search extension provides semantic search over local text and markdown files. It indexes configured directories using vector embeddings, syncs on session startup via a child process, and exposes a [[extension-interface#knowledge_search Tool]] to the LLM.

## Overview

The extension follows pi's lifecycle model with three phases: initialization, indexing, and search. On session start, it loads [[configuration]], creates an embedder via [[embedding]], and spawns a background [[sync-worker]]. Search queries use [[index-store]] with optional [[kb-searcher]] results.

## Component Graph

Source modules and their roles within the extension.

- [[src/index.ts|index.ts]] — Entry point. Registers commands and tools with pi, manages lifecycle.
- [[src/config.ts|config.ts]] — Loads/saves configuration with env var overrides.
- [[src/embedder.ts|embedder.ts]] — Embedding abstraction over OpenAI, Bedrock, and Ollama.
- [[src/chunker.ts|chunker.ts]] — Markdown-aware content chunking for indexing.
- [[src/index-store.ts|index-store.ts]] — Persistent JSON index with cosine similarity search.
- [[src/sync-worker.ts|sync-worker.ts]] — Forked child process that performs background sync.
- [[src/kb-searcher.ts|kb-searcher.ts]] — Bedrock Knowledge Base integration (optional).

## Session Lifecycle

1. **session_start**: Load config, create embedder + index, load persisted index from disk, spawn sync worker.
2. **Sync worker** (child process): Performs incremental or full re-index in background. Writes JSON result to stdout, then exits.
3. **session_shutdown**: Signals worker exit, flushes pending index saves.

File watching was removed (commit d38a81f) because it caused UI freezes. The extension relies on sync-on-startup only — users re-index via `/knowledge-reindex` or by restarting the session.

## Data Flow

Files on disk → [[chunking]] → text chunks → [[embedding]] → vectors → [[index-store]] stores entries keyed by `absPath#chunkIndex`. At search time, the query is embedded and compared against all stored vectors using [[index-store#Search|cosine similarity]].

## Deployment

The sync worker is pre-compiled with esbuild to avoid ESM/CJS compatibility issues with tsx on Node 25+. Build command: `npm run build:worker`. The compiled output lives at `dist/sync-worker.mjs`.
