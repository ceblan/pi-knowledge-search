# Chunking

The chunker splits markdown content into semantically meaningful pieces for embedding. It uses heading-aware splitting with paragraph fallback and size-based hard-splitting.

## Strategy

[[src/chunker.ts#chunkMarkdown|chunkMarkdown]] implements a multi-pass splitting strategy:

1. Split on `##` level-2+ headings (NOT on `#` level-1 headings)
2. Each section becomes a chunk candidate
3. Sections exceeding `maxChunkSize` are split on double-newline paragraphs
4. Remaining oversized chunks are hard-split at `maxChunkSize` with overlap
5. Chunks smaller than `minChunkSize` are merged with neighbors

If the entire file fits within `maxChunkSize`, it is returned as a single chunk.

## Heading Splitting

[[src/chunker.ts#splitByHeadings|splitByHeadings]] matches `#{2,6}` heading patterns. Content before the first heading gets the "intro" label. Level-1 (`#`) headings are intentionally excluded from splitting — they typically represent the document title and should remain with the intro content.

## Paragraph Splitting

[[src/chunker.ts#splitByParagraphs|splitByParagraphs]] splits on `\n\n+` boundaries. Paragraphs are accumulated until adding the next one would exceed `maxChunkSize`, then the current accumulation is flushed as a chunk.

## Hard Splitting

[[src/chunker.ts#hardSplit|hardSplit]] is the last resort for oversized chunks. It slices at `maxChunkSize` with configurable overlap (default 200 chars) to preserve context at boundaries. Includes an infinite-loop guard.

## Tiny Chunk Merging

[[src/chunker.ts#mergeTiny|mergeTiny]] prevents very small chunks from cluttering the index. Chunks below `minChunkSize` are merged with adjacent chunks if the combined size stays within `maxChunkSize`. When a preceding chunk is tiny, it adopts the next chunk's heading.

## Chunk Metadata

Each [[src/chunker.ts#Chunk|Chunk]] carries `heading` (section label or "intro"), `startLine` (0-indexed line number), and `charOffset` (character position in the original content). These enable precise source mapping in search results.

## Defaults

Default `maxChunkSize` is 3,000 characters and `minChunkSize` is 200 characters. Both are configurable via function parameters.
