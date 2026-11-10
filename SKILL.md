---
name: lean-code-paths
description: Use when writing, modifying, refactoring, or reviewing code. Trace a caller's operation through its API, application code, middleware, and database to cut redundant validation, round trips, and data movement.
license: MIT
---

# Lean Code Paths

Start with one real operation. What does the caller display or act on? Follow that need through the API, service, middleware, ORM, and executed SQL. Eliminate unnecessary work at the earliest point on the affected path. Use straightforward, low-cost code in routine development. Locate the bottleneck before introducing a more complex optimization.

## Implementation rules

- Reuse data already available to the request. Batch queries, writes, and remote calls whenever the operation supports it. Split batches at actual limits. Check what a bulk API does internally. Parallel per-item calls still incur one round trip per item.
- Choose data structures for the access pattern. Scan an existing collection for a single lookup. Build a keyed map once for repeated matches. Move invariant parsing and computation out of loops. On hot paths, eliminate repeated sorts, large copies, and unused intermediate collections. Pass existing values within a process instead of round-tripping through JSON to convert between identical shapes.
- Parse and validate external input where it enters. Internal functions can use values that the entry point has already checked for presence, type, or range. Before adding a defensive branch, identify a reachable input and the failure the branch prevents. Catch an error where the code can recover, add useful context, or translate it into a different error contract.
- Include ownership in the read or write predicate where possible. Pass verified task identity, ownership, and arguments through trusted modules, schedulers, and executors. A handoff, callback, or delegation within the same task does not by itself require another check or lookup. When a new input or independent trust boundary is involved, verify the guarantees the receiver cannot inherit. Enforce mutable conditions such as balance, prior state, and idempotency atomically at the write that depends on them.
- Use a suitable framework default or define a fixed rule once where it belongs. Add a configuration setting only for a real deployment difference, independent tuning need, or explicit user request. Use the existing configuration or secret store first. Create a file only when the existing mechanism cannot serve the requirement. Add feature switches only when requested.
- Prefer unique constraints, conditional writes, atomic increments, and short transactions. Before adding an application lock, identify the competing operations, the incorrect outcome, and why existing mechanisms cannot prevent it. Lock the smallest resource for the shortest interval. Check every writer and any late result before removing an existing lock.
- Preserve access rules, ordering, pagination, partial-success semantics, output quality, and recovery. Discuss material changes to behavior or cost with the user. Reuse the project's middleware, test entry points, and acceptance criteria.

## Read the relevant reference

- For SQL, ORM behavior, indexes, bulk operations, transactions, and locking, read [Database access and transactions](references/database.md).
- For API field selection, middleware, caching, queues, large files, or frontend loading, read the relevant section of [Runtime and middleware](references/runtime.md).

## Verification

- Exercise changed APIs through the actual application entry point in an authorized test environment. Check the response, affected callers, and final business state. When trimming fields, inspect the actual payload and, for database-backed paths, the executed SQL. For batching, compare query counts for a small batch and a full one. Test the relevant race, duplicate request, or recovery path when side effects are involved. Verify background jobs and local computation through their normal entry points and inspect their outputs. Follow required project checks and CI.
- The user owns visual and manual web acceptance. Hand over the page, actions, and expected results, and mark that acceptance as pending. Run the relevant build, type, or interaction checks for frontend changes. If the user delegates browser testing, cover the requested scope.
- Support speedup claims with before-and-after measurements in the same environment under equivalent data and concurrency. Record sample count and cache state. Choose metrics that expose the cost under investigation, such as tail latency, throughput, query count, rows scanned, bytes transferred, or peak memory. For asynchronous work, distinguish acceptance, queue delay, and completion. Report a single run as a single run. Without a comparable baseline, report only what was measured. Keep verification proportional to the change.
