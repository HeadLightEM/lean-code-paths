# Runtime and middleware

## API fields and callers

- Accept fields the caller may set and the operation needs. Take the acting identity from authenticated context. A resource ID in the route need not appear again in the body. For partial updates, write only authorized fields that were supplied. Preserve the distinction between omission and explicit clearing. Bind input to the intended fields rather than an entire persistence entity.
- Select response fields explicitly for the caller's permissions and its display, actions, and pagination. Return a fixed summary for lists and load details when opened. Include data every visible item needs in the batch response so the caller can render the page without a detail request per row.
- Before trimming a shared or published API, check real consumers and promised contracts, including exports, jobs, and supported clients. Reuse an existing summary where practical. Introduce views, dynamic field selection, or a new endpoint only when consumers have genuinely different needs and the saved work justifies the added contract. Update the existing API definition, types, and affected callers together.

For example, an asset picker displays a name and thumbnail, submits an ID when selected, and uses a cursor to load more. Its response can contain `items[{id, name, thumbnailUrl}]` and `nextCursor`. Filter by the authenticated account in the query and read the columns needed for the summary and cursor. Skip original file URLs, parser reports, and full file details. If an export also consumes that summary, its required fields count too.

## Middleware and clients

- Trace authentication, filters, interceptors, business code, and external calls for one request. Share request identity and fetched data. Eliminate repeated user lookups, repeated parsing of the same JSON, and full-body logging. Keep independent trust-boundary checks and required audit events. Use existing stage timings to locate delays.
- Reuse HTTP, database, and storage clients within their supported lifetimes. Return connections, set timeouts, and release response bodies according to the client's contract. Respect process and thread ownership.
- Before adding Redis, a queue, a search engine, a CDN, a service, or an in-process cache, identify the work it removes, the round trips or consistency costs it adds, and how to verify the tradeoff. Perform cheap local computations directly when they need neither sharing nor persistence.

## Caches and delivery

- Reuse request-local results first. For cross-request caching, establish reuse frequency, key scope, tenant isolation, version, capacity, expiry, and invalidation. Choose acceptable staleness for the operation. Use authoritative state for charging, final authorization, and capacity reservations. Publish or invalidate affected entries after commit.
- Measure hits, misses, origin load, and invalidation cost. Coalesce concurrent misses when an actual hot key warrants it. Use suitable Redis bulk commands or bounded pipelines for grouped operations. Pipelining reduces round trips and provides no guarantee of business atomicity.
- For cacheable public versioned assets, use stable URLs, HTTP caching, and an existing CDN, with freshness and revalidation matched to the update strategy. Preserve access rules and link expiry for private results. When authorized direct upload or download already exists, use it for large files. Keep authorization, registration, and completion checks with their owning service.

## Concurrency, queues, and retries

- Separate CPU work, I/O, connection waits, and queue delay. Parallelize independent work using existing executors. Bound concurrency by the tightest CPU, memory, database, or downstream limit. Consider worker, thread, and connection counts together. Compare throughput, tail latency, errors, and utilization before increasing concurrency.
- Use queues for a real asynchronous delivery or load-leveling need. Bound queues and batches. Apply the agreed backpressure or rejection behavior when consumers fall behind. Observe oldest-item wait and end-to-end completion. Claim eligible work incrementally in stable order using state and due time. Preserve late arrivals, lease expiry, and interruption recovery.
- Consolidate polling, stop at terminal states, and release resources when idle. Reuse reliable notifications where available. Set timeouts suitable for the task and cap backoff for retryable transient failures. Keep retry ownership in one layer where possible. Check the original record before repeating paid operations or writes with an unknown outcome.

## Computation, images, and large files

- Evaluate algorithmic cost, peak memory, and data movement at realistic scale. Remove repeated decoding, full-image compositing, encoding, downloads, disk I/O, and large copies. Avoid repeated conversions between binary, Base64, JSON, databases, and services. Prefer bounded streams, chunks, file references, or supported direct transfer paths.
- Match content version, dimensions, color, and composition settings before reusing an output. Generate previews at display size and preserve final delivery quality. Confirm that recovery artifacts are durable before releasing the last usable copy. Release handles, buffers, temporary files, and GPU resources on completion, failure, and cancellation.
- Compare representative inputs and whole-batch peak use. Verify output equivalence, timings, and resource consumption before changing languages, codecs, acceleration, or concurrency. A single-core example or single-stage benchmark cannot establish end-to-end production gains.

## Frontend requests and rendering

- Remove request waterfalls, per-card lookups, and redundant refreshes. Share server state and completed results. Prevent stale responses from replacing a newer filter or account. Refresh the affected data, coalesce frequent interactions and polls, and keep first paint dependent only on essential resources.
- Load appropriately sized images, reuse thumbnails and native caching, and defer expensive offscreen content. Use measurements to prioritize bundle size, list rendering, subscription scope, repeated computation, or obsolete paint work. Introduce code splitting, virtualization, workers, or memoization where the measured benefit warrants it.
