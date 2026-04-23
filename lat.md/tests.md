---
lat:
  require-code-mention: true
---
# Tests

Unit test specifications for configuration, chunking, and index-store modules.

## Config Tests

Tests for the [[configuration]] system covering loading, env overrides, defaults, and error cases.

### getConfigPath returns env-configured path

Verifies that [[src/config.ts#getConfigPath|getConfigPath]] returns the path set via `KNOWLEDGE_SEARCH_CONFIG` env var.

### Returns null when no config file and no env vars

Ensures [[src/config.ts#loadConfig|loadConfig]] returns null when neither a config file nor fallback env vars are present.

### Loads valid config from file

Verifies that a well-formed JSON config file is correctly parsed into a [[src/config.ts#Config|Config]] object with all fields populated.

### Returns null for corrupt JSON config file

Ensures corrupted JSON files are handled gracefully by returning null rather than throwing.

### Uses env var KNOWLEDGE_SEARCH_DIRS as fallback

Verifies that `KNOWLEDGE_SEARCH_DIRS` env var provides directories and triggers OpenAI provider via `OPENAI_API_KEY` when no config file exists.

### Applies default values for optional fields

Confirms that omitted optional fields (fileExtensions, excludeDirs, dimensions) get sensible defaults.

### Resolves tilde in directory paths

Ensures `~` prefix in directory paths is expanded to `$HOME`.

### Throws for openai provider without API key

Verifies that loading config with an OpenAI provider but no API key throws a descriptive error.

### Configures bedrock provider

Tests Bedrock provider configuration with profile, region, and model fields.

### Configures ollama provider

Tests Ollama provider configuration with URL and model fields.

### Throws for unknown provider type

Verifies that an unrecognized provider type triggers an error.

### Env vars override config file values

Confirms that env vars take precedence over config file values for dirs and dimensions.

### Env var overrides provider API key

Ensures `KNOWLEDGE_SEARCH_OPENAI_API_KEY` overrides the API key from the config file.

### saveConfig writes valid JSON to config path

Verifies that [[src/config.ts#saveConfig|saveConfig]] writes a valid JSON file to the configured path.

### Returns null when dirs resolve to empty

Ensures config returns null when dirs array is empty and no KBs are configured.

### Bedrock provider uses defaults when fields missing

Confirms Bedrock provider gets default profile, region, and model when only type is specified.

### Ollama provider uses defaults when fields missing

Confirms Ollama provider gets default URL and model when only type is specified.

## Chunker Tests

Tests for the [[chunking]] system covering edge cases, heading splitting, paragraph splitting, and size constraints.

### Returns empty array for empty string

Verifies [[src/chunker.ts#chunkMarkdown|chunkMarkdown]] returns an empty array for empty input.

### Returns empty array for whitespace-only string

Ensures whitespace-only input produces no chunks.

### Returns single chunk for short content

Verifies that content shorter than maxChunkSize is returned as a single chunk with "intro" heading.

### Splits on ## headings

Confirms that level-2 headings produce separate chunks with correct heading labels.

### Assigns intro heading for content before first heading

Ensures content preceding the first heading gets the "intro" heading label.

### Handles markdown with no headings (paragraphs only)

Verifies that files without headings are split on paragraphs, all retaining "intro" heading.

### Hard-splits a very long single paragraph

Ensures oversized content is split into multiple chunks with full text coverage.

### Hard-split chunks have overlap

Verifies that hard-split chunks stay within maxSize and overlapping content is preserved.

### Preserves code blocks in chunks

Ensures fenced code blocks remain intact within their chunks.

### Merges tiny chunks with neighbors

Verifies that sections below minChunkSize are merged with adjacent chunks rather than kept standalone.

### Tracks startLine correctly across sections

Ensures startLine metadata is accurate for the first chunk.

### Tracks charOffset correctly

Verifies that charOffset increases for subsequent chunks.

### Handles level 3-6 headings as section breaks

Confirms that heading levels 3 through 6 trigger section splits like level 2.

### Does NOT split on level 1 headings

Verifies that `#` headings do not cause section breaks — content stays in "intro" section.

### Respects custom maxChunkSize

Ensures all chunks respect a custom maxSize parameter.

### Respects custom minChunkSize for merging

Verifies that a higher minChunkSize produces fewer merged chunks.

## Index Store Tests

Tests for the [[index-store]] utility functions.

### dotProduct returns 0 for orthogonal vectors

Verifies [[src/index-store.ts#dotProduct|dotProduct]] returns 0 for perpendicular vectors.

### dotProduct returns 1 for identical unit vectors

Confirms dot product of a unit vector with itself is approximately 1.0.

### dotProduct returns -1 for opposite unit vectors

Verifies dot product of opposite unit vectors returns -1.

### dotProduct computes correct value

Confirms the arithmetic: dotProduct([1,2,3], [4,5,6]) = 32.

### dotProduct handles empty vectors

Ensures empty vector inputs return 0 without errors.

### dotProduct handles mismatched lengths

Verifies that vectors of different lengths use the shorter length.

### dotProduct works with high-dimensional vectors

Confirms correct computation for 512-dimensional vectors (typical embedding size).
