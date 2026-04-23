# KB Searcher

The [[src/kb-searcher.ts#BedrockKBSearcher|BedrockKBSearcher]] provides search integration with AWS Bedrock Knowledge Bases. It is an optional dependency — the AWS SDK is lazily imported so the extension works without it.

## Initialization

[[src/kb-searcher.ts#BedrockKBSearcher|BedrockKBSearcher]] lazily initializes on first search via [[src/kb-searcher.ts#BedrockKBSearcher#_init|_init]]. It creates `BedrockAgentRuntimeClient` instances using AWS credential profiles. Clients are reused across KBs that share the same region and profile combination.

If the AWS SDK fails to load (e.g., not installed), initialization sets configs to empty and all subsequent searches return no results — graceful degradation.

## Search

[[src/kb-searcher.ts#BedrockKBSearcher#search|search]] queries each configured KB in parallel using the `RetrieveCommand` API. Results are normalized to the same [[src/index-store.ts#SearchResult|SearchResult]] shape used by local index results. The score threshold is 0.15, matching the local index threshold.

## Result Merging

In [[extension-interface#knowledge_search Tool]], local index and KB results are merged, sorted by score, and truncated to the requested limit. KB results include a label suffix (e.g., `[My KB]` or `[KB]`) in the path field for identification.

## Supported Locations

Bedrock KB results may originate from various data sources. The searcher extracts URIs from S3, web, Confluence, Salesforce, and SharePoint locations. Unknown locations display "unknown" as the path.

## Configuration

KBs are configured in the `knowledgeBases` array in the config file. Each entry needs an `id` (Bedrock KB ID) and optional `region`, `profile`, and `label` fields. The `/knowledge-add-kb` command provides an interactive setup wizard.
