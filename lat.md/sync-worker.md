# Sync Worker

The sync worker is a forked child process that performs background indexing without blocking the main event loop. It loads config, creates an embedder and index, runs sync, and reports results via stdout.

## Process Model

[[src/sync-worker.ts]] is pre-compiled to `dist/sync-worker.mjs` using esbuild. The main process spawns it via `fork()` with piped stdio. It writes JSON to stdout on success and exits. Errors go to stderr.

The pre-compilation avoids ESM/CJS compatibility issues between tsx and Node 25+. Rebuild with: `npm run build:worker`.

## Result Format

On successful sync, the worker writes a single JSON line to stdout:
```json
{"added": 5, "updated": 2, "removed": 1, "size": 42, "chunks": 187}
```

The main process parses this, reloads the index from disk (since the worker updated it), and displays a status message via `ctx.ui.setStatus` that auto-clears after 5 seconds.

## Worker Lifecycle

The main process manages the worker with a restart policy: up to 3 restarts within a 60-second window. If the worker crashes more than 3 times, it gives up. The `workerExitExpected` flag prevents restart attempts during intentional shutdown.

## Error Handling

The worker installs handlers for `uncaughtException` and `unhandledRejection` that write error messages to stderr and exit with code 1. This ensures the parent process always gets a clean exit signal rather than hanging.
