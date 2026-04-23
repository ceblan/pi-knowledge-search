# Configuration

The configuration system loads settings from a JSON file and applies environment variable overrides. It supports three embedding providers and optional Bedrock Knowledge Base integrations.

## Config File

Configuration is stored at `~/.pi/knowledge-search.json` by default, overridable via `KNOWLEDGE_SEARCH_CONFIG`. The [[src/config.ts#ConfigFile|ConfigFile]] interface defines the raw JSON shape, while [[src/config.ts#Config|Config]] is the fully resolved output with defaults applied.

Key fields: `dirs` (directories to index), `fileExtensions`, `excludeDirs`, `dimensions` (vector size, default 512), `provider` (embedding provider config), `knowledgeBases` (optional Bedrock KBs).

## Environment Variable Overrides

Environment variables take precedence over config file values. The precedence chain is: env var → config file → built-in default. The [[src/config.ts#loadConfig|loadConfig]] function implements this layering.

`KNOWLEDGE_SEARCH_DIRS` is a special fallback — if no config file exists but this env var is set, the extension initializes with those directories and an OpenAI provider using `OPENAI_API_KEY`.

## Provider Types

Three provider types are supported, each with provider-specific env vars:

- **openai**: Requires API key via `OPENAI_API_KEY` or `KNOWLEDGE_SEARCH_OPENAI_API_KEY`. Default model: `text-embedding-3-small`.
- **bedrock**: Uses AWS credentials from the named profile. Default model: `amazon.titan-embed-text-v2:0`.
- **ollama**: Local embedding server. Default URL: `http://localhost:11434`, default model: `nomic-embed-text`.

If the provider type is unknown, [[src/config.ts#loadConfig|loadConfig]] throws an error. OpenAI provider without an API key also throws.

## Path Resolution

Directory paths in the config file support `~` prefix expansion via `HOME`. This happens in [[src/config.ts#loadConfig|loadConfig]] using a simple `replace(/^~/, home)` transform.

## Defaults

When optional fields are omitted: fileExtensions defaults to `[.md, .txt]`, excludeDirs to `[node_modules, .git, .obsidian, .trash]`, dimensions to `512`, and indexDir to `~/.pi/knowledge-search`.
