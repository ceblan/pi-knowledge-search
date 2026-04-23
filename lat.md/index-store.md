# Index Store

The [[src/index-store.ts#KnowledgeIndex|KnowledgeIndex]] class manages a persistent JSON-based vector index. It stores chunk-level entries with embeddings, supports incremental sync and full rebuild, and provides cosine similarity search with file-level deduplication.

## Storage Format

The index is stored as `index.json` in the configured index directory. The [[src/index-store.ts#IndexData|IndexData]] structure contains a version, dimensions, and entries keyed by `absPath#chunkIndex`. Version 3 is current; mismatches trigger a fresh index.

Each [[src/index-store.ts#IndexEntry|IndexEntry]] stores: `relPath`, `sourceDir`, `mtime`, `vector` (embedding), `excerpt` (capped at 3,500 chars), `heading`, and `chunkIndex`.

## Sync

[[src/index-store.ts#KnowledgeIndex#sync|sync]] performs incremental indexing by scanning directories, comparing mtimes, chunking changed files via [[chunking]], and embedding in batches of 50. Removed files are cleaned up.

## Search

[[src/index-store.ts#KnowledgeIndex#search|search]] embeds the query, computes [[src/index-store.ts#dotProduct|dot product]] against all vectors, sorts by score, and deduplicates to one result per file. Results below 0.15 threshold are filtered.

## File Operations

Methods for updating and removing individual files from the index.

- [[src/index-store.ts#KnowledgeIndex#updateFile|updateFile]] re-indexes a single file: removes old chunks, embeds new chunks, schedules a debounced save.
- [[src/index-store.ts#KnowledgeIndex#removeFile|removeFile]] deletes all chunks for a file path and schedules save.
- [[src/index-store.ts#KnowledgeIndex#rebuild|rebuild]] clears all entries then runs a full sync.

## Save Debouncing

[[src/index-store.ts#KnowledgeIndex#scheduleSave|scheduleSave]] batches writes with a 5-second debounce timer. Multiple rapid updates (e.g., during sync) coalesce into a single disk write. [[src/index-store.ts#KnowledgeIndex#close|close]] flushes any pending save immediately.

## File Scanning

[[src/index-store.ts#KnowledgeIndex#walkDir|walkDir]] recursively scans configured directories, filtering by file extension and skipping excluded directories and hidden directories (starting with `.`). YAML frontmatter is stripped from file content before chunking.

## Score Threshold

Search results below a cosine similarity of 0.15 are excluded. This threshold prevents low-relevance noise from appearing in LLM context.
