# Embedding

The embedding subsystem converts text into fixed-dimension vector representations. A factory function creates provider-specific embedder instances that share a common [[src/embedder.ts#Embedder|Embedder]] interface.

## Embedder Interface

The [[src/embedder.ts#Embedder|Embedder]] interface defines two methods: `embed` for a single text and `embedBatch` for multiple texts with configurable concurrency. All implementations accept an optional `AbortSignal` for cancellation.

## Provider Implementations

[[src/embedder.ts#createEmbedder|createEmbedder]] is the factory that returns the correct implementation based on provider config:

- **[[src/embedder.ts#OpenAIEmbedder|OpenAIEmbedder]]**: Uses the OpenAI `/v1/embeddings` API with native batch support (up to 2048 inputs). Internally chunks into batches of 100 for payload safety.
- **[[src/embedder.ts#BedrockEmbedder|BedrockEmbedder]]**: Uses the AWS Bedrock Runtime `InvokeModelCommand`. The AWS SDK is lazily imported to avoid hard dependency. Uses parallel mapping with concurrency 10.
- **[[src/embedder.ts#OllamaEmbedder|OllamaEmbedder]]**: Calls the local Ollama `/api/embed` endpoint. Accepts an optional `dimensions` parameter which is forwarded in the API request body, enabling models like `qwen3-embedding` to return user-defined vector sizes (32–4096). Uses parallel mapping with concurrency 4. Falls back to individual calls since batch support varies by model.

## Rate Limit Handling

All providers use [[src/embedder.ts#withRateLimitRetry|withRateLimitRetry]] which catches HTTP 429 responses (and Bedrock `ThrottlingException`) and retries with exponential backoff at 1s, 2s, 4s delays. Maximum 3 retries before propagating the error.

## Text Truncation

[[src/embedder.ts#truncate|truncate]] caps input text at 10,000 characters (approximately 4-6K tokens) to stay within provider token limits. This is applied before sending text to the embedding API.

## Batch Processing

[[src/embedder.ts#parallelMap|parallelMap]] provides bounded-concurrency parallel execution. It spawns workers that pull from a shared cursor, respecting abort signals. Default concurrency: OpenAI uses native batching, Bedrock 10, Ollama 4.
