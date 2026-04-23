# Extension Interface

The extension registers with pi via [[src/index.ts]] and exposes three slash commands and one LLM tool. It integrates [[configuration]], [[embedding]], [[index-store]], [[sync-worker]], and [[kb-searcher]] into pi's session lifecycle.

## Lifecycle Hooks

The extension integrates with pi's session lifecycle to manage initialization and cleanup.

### session_start

On session start, the extension loads [[configuration]] via [[src/config.ts#loadConfig|loadConfig]]. If configured, it creates an embedder, initializes [[src/index-store.ts#KnowledgeIndex|KnowledgeIndex]], loads the persisted index from disk, and spawns a [[sync-worker]] child process. If Bedrock Knowledge Bases are configured, it creates a [[src/kb-searcher.ts#BedrockKBSearcher|BedrockKBSearcher]] instance.

In KB-only mode (no local provider configured), the extension skips local indexing and sets `syncDone = true` immediately.

### session_shutdown

On shutdown, the extension sets the `workerExitExpected` flag to prevent worker restarts, and calls [[src/index-store.ts#KnowledgeIndex#close|close]] to flush pending index saves.

## Commands

Three slash commands provide interactive configuration and index management.

### /knowledge-search-setup

Interactive wizard that prompts for directories, file extensions, exclude patterns, and embedding provider (OpenAI, Bedrock, or Ollama) with provider-specific configuration. Saves to [[configuration]] and notifies the user to run `/reload`.

### /knowledge-add-kb

Adds a Bedrock Knowledge Base to the config. Prompts for KB ID, label, region, and profile. Loads existing config, appends the KB entry (rejecting duplicates), and saves.

### /knowledge-reindex

Triggers a full re-index of all configured directories via [[src/index-store.ts#KnowledgeIndex#rebuild|rebuild]]. Reports final file and chunk counts on completion.

## knowledge_search Tool

The `knowledge_search` tool provides semantic search over indexed content. It accepts a `query` string and optional `limit` (default 8, max 20). Local index and KB results are searched in parallel, merged by score, and deduplicated per file.

The tool returns empty results with an informative message if: the extension is not configured, the index is still syncing, or the index is empty. Each result displays the file path (with `~` home substitution), heading context, match percentage, and content excerpt.
