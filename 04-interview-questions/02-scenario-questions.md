# Scenario & Troubleshooting Questions (with model answers)

Interviewers increasingly ask "what would you do if..." questions. A strong answer: **clarify -> find the bottleneck with data -> short-term mitigation -> long-term fix -> prevention.**

---

### 1. Your database CPU is at 100% and the site is slow. What do you do?
- **Diagnose:** slow query log / `pg_stat_statements` -> top queries by total time; check missing indexes, N+1 queries, lock contention, a recent deploy.
- **Mitigate now:** kill runaway queries, roll back the deploy, enable/raise caching, shed non-critical traffic, scale up if possible.
- **Fix:** add indexes, rewrite queries, add read replicas for read traffic, cache hot reads, move analytics to a replica/warehouse.
- **Prevent:** query reviews, load tests, alerts on DB saturation.

### 2. p99 latency doubled after a deploy, but p50 is fine.
Tail issue: GC pauses, a slow dependency on some code path, lock contention, cold caches, a bad host. Compare traces of slow requests; roll back if correlated with the deploy; check per-host metrics (one bad instance?). Use canaries next time.

### 3. A cache cluster node died and the DB is now overloaded.
Classic stampede. Short term: rate-limit/shed, serve stale data, bring up replacement and warm it. Long term: replicas for cache shards, consistent hashing (only 1/N keys miss), request coalescing, circuit breaker in front of the DB.

### 4. A celebrity just posted and the timeline service is falling over.
Fan-out storm. Switch the account to pull-based delivery (hybrid fan-out), queue fan-out work, cache the celebrity's recent posts aggressively.

### 5. Users report seeing old data right after updating their profile.
Read-after-write inconsistency due to replica lag or stale cache. Read user's own profile from the primary for a short window after writes; invalidate cache on write (delete key); version-check cache entries.

### 6. Payments are occasionally charged twice.
Retries without idempotency. Add idempotency keys on client -> API -> PSP; unique constraint on payment order ID; treat timeouts as unknown and query PSP status before retrying; reconcile daily.

### 7. Kafka consumer lag keeps growing.
Consumers too slow or too few. Check processing time per message, downstream bottlenecks, poison messages (send to DLQ), rebalancing loops. Scale consumers (≤ partitions), increase partitions for future, batch processing, optimize downstream writes.

### 8. One shard of the database is much hotter than the others.
Skewed shard key or hot entity. Short term: cache hot keys, add read replicas for that shard. Long term: split the hot partition, re-key with salting, move a big tenant to a dedicated shard.

### 9. Your service depends on a third-party API that's flaky.
Timeouts, retries with backoff + jitter (idempotent calls only), circuit breaker, fallback (cached/default response), bulkhead its thread pool, queue requests for later where possible, monitor its SLA.

### 10. Traffic will spike 20× for a planned sale tomorrow.
Pre-scale (servers, DB replicas, caches), load-test, pre-warm caches/CDN, enable waiting room for the hot path, feature-flag off non-critical features, freeze deploys, staff on-call, prepare rollback and kill switches.

### 11. A region is going down. How do you fail over?
Health checks fail -> DNS/anycast routes traffic to healthy region(s); promote replicas there (accept RPO for async data); ensure capacity headroom; communicate status. Practice with game days.

### 12. Disk usage on log servers grows 30% per month.
Retention tiers (7 days hot, then S3), compression, sampling of DEBUG logs, drop noisy logs, per-team quotas, move metrics-like logs to metrics.

### 13. Search results don't show newly added products for minutes.
Indexing pipeline lag (CDC -> Kafka -> indexer -> refresh interval). Check consumer lag, refresh interval, bulk sizes; for critical updates, write-through to the index or show "processing" state.

### 14. Your WebSocket servers can't be deployed without dropping users.
Graceful drain: stop new connections, send reconnect hints, let clients reconnect with jitter to other servers, resync state by last message ID.

### 15. An API key is leaked publicly.
Revoke/rotate immediately, audit usage logs, rate-limit/block abusive traffic, notify affected parties, add secret scanning in CI and repos.

### 16. Your job scheduler ran a nightly billing job twice.
Two schedulers both claimed it (no lease/fencing) or a retry after a timeout. Use leases with fencing, conditional status transitions, and make the job idempotent (per-period unique key).

### 17. The recommendation service is slow and makes the home page slow.
Set a strict timeout; fall back to popular/cached recommendations; render the page without that module (graceful degradation); fix root cause separately.

### 18. Costs on the CDN bill tripled.
Check cache hit ratio (cache key with random query params?), bigger media files (missing compression/resizing), bot traffic, hotlinking. Normalize cache keys, compress, resize, block abusive referrers.

### 19. How would you migrate a large table to a new schema with zero downtime?
Expand/contract: add new columns/table -> dual-write -> backfill in batches -> verify -> switch reads -> stop old writes -> drop old structure. Use feature flags and keep rollback possible.

### 20. How would you move from a monolith DB to sharded DBs live?
Introduce a data access layer, pick shard key, copy data per shard with CDC catch-up, dual-read verification, cut over per tenant/range, keep reverse replication for rollback.

---

**See also:** [Reliability patterns](../01-fundamentals/13-reliability-and-resilience-patterns.md) · [Observability](../01-fundamentals/14-observability-logs-metrics-tracing.md)
