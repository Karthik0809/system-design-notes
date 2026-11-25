# Concept Questions with Answers (100)

Short, interview-ready answers. Each topic links to the full note. Try answering out loud before reading the answer.

---

## Scaling & architecture
1. **Vertical vs horizontal scaling?** Vertical = bigger machine (simple, has a ceiling, SPOF). Horizontal = more machines behind a load balancer (near-unlimited, needs stateless design and data partitioning).
2. **Why make web servers stateless?** Any server can handle any request -> easy autoscaling, rolling deploys and failover. Session state goes to Redis/DB.
3. **What is a single point of failure and how do you remove it?** A component whose failure stops the system; add redundancy (replicas, LB pairs, multi-AZ) and automatic failover.
4. **Monolith or microservices first?** Usually a modular monolith; split into services when team size, independent scaling or deploy cadence demands it.
5. **What is an API gateway for?** Single entry point: auth, rate limiting, routing, request aggregation, protocol translation, observability.
6. **What is a service mesh?** Sidecar proxies (Envoy) handling mTLS, retries, timeouts, load balancing and telemetry for service-to-service calls.
7. **What is cell-based architecture?** Independent full-stack copies ("cells") each serving a subset of users -> limits blast radius.
8. **Sync vs async communication between services?** Sync (REST/gRPC) is simple but couples availability and latency; async (events) decouples but brings eventual consistency.

## Networking & APIs
9. **TCP vs UDP?** TCP: reliable, ordered, connection-oriented. UDP: fast, no guarantees - real-time media, gaming, DNS.
10. **What happens when you type a URL?** DNS -> TCP handshake -> TLS handshake -> HTTP request -> LB -> app -> response -> render (+ CDN assets).
11. **HTTP/2 vs HTTP/3?** HTTP/2 multiplexes streams over one TCP connection; HTTP/3 runs over QUIC (UDP) to remove TCP head-of-line blocking and speed up handshakes.
12. **WebSocket vs SSE vs long polling?** WebSocket = bidirectional persistent; SSE = server->client stream over HTTP; long polling = held HTTP requests (fallback).
13. **REST vs gRPC vs GraphQL?** REST: public, cacheable. gRPC: fast binary internal RPC with streaming. GraphQL: client chooses fields; good for aggregating many backends.
14. **Offset vs cursor pagination?** Offset is slow for deep pages and unstable under inserts; cursor (`WHERE id < last_id`) is fast and stable.
15. **What is idempotency and why does it matter?** Repeating an operation has the same effect as once; needed because clients retry after timeouts. Use idempotency keys.
16. **401 vs 403?** 401 = not authenticated; 403 = authenticated but not allowed.
17. **502 vs 504?** 502 = bad response from upstream; 504 = upstream timed out.
18. **What is a CDN and when do you use it?** Edge caches near users for static/media (and some dynamic) content -> lower latency and origin load.

## Load balancing
19. **L4 vs L7 load balancer?** L4 routes on IP/port (fast); L7 understands HTTP (path routing, headers, cookies, TLS termination).
20. **Which LB algorithm for WebSockets?** Least connections (connections are long-lived and uneven).
21. **What is "power of two choices"?** Pick two random servers, send to the less loaded - near-optimal without global state.
22. **How do health checks work?** LB probes `/health`; failing servers are removed; passive outlier detection on real traffic.
23. **How do you avoid the LB being a SPOF?** Active-passive pair with floating IP, or anycast/DNS across many LBs; managed cloud LBs are already redundant.

## Databases
24. **When SQL vs NoSQL?** SQL for transactions, joins, strong consistency; NoSQL for massive scale, flexible schema, simple key-based access patterns.
25. **Explain ACID.** Atomicity (all-or-nothing), Consistency (constraints hold), Isolation (concurrent txns don't interfere), Durability (committed data survives crashes).
26. **Isolation levels?** Read uncommitted < read committed < repeatable read < snapshot < serializable; higher = fewer anomalies, less concurrency.
27. **What is write skew?** Two transactions read the same data and update different rows, jointly breaking an invariant; prevented by serializable isolation or explicit locks.
28. **Optimistic vs pessimistic locking?** Optimistic: version check at write, retry on conflict (low contention). Pessimistic: lock rows up front (`SELECT FOR UPDATE`) (high contention).
29. **B-tree vs LSM tree?** B-tree: fast reads, in-place updates. LSM: very fast sequential writes, reads may check several SSTables (Bloom filters help).
30. **What is a covering index?** An index that contains all columns a query needs -> no table lookup.
31. **Composite index leftmost-prefix rule?** Index (a, b, c) serves filters on a, (a, b), (a, b, c) - not b alone.
32. **Why denormalize?** Avoid joins (especially across shards) for faster reads, at the cost of duplicated data and harder updates.
33. **What is MVCC?** Multiple row versions so readers see a consistent snapshot without blocking writers.
34. **What is connection pooling?** Reusing DB connections across requests; proxies like PgBouncer let thousands of app instances share few connections.

## Replication & partitioning
35. **Leader-follower replication trade-offs?** Simple, no write conflicts; leader is a write bottleneck and failover can lose async writes.
36. **Sync vs async replication?** Sync = no data loss, higher latency; async = fast, may lose recent writes on failover; semi-sync = one follower confirms.
37. **What is replication lag and its problems?** Followers behind the leader -> read-your-writes and monotonic-read anomalies; fix by reading own writes from leader / sticky replicas.
38. **What is split brain?** Two nodes both think they're leader; prevent with quorum-based election and fencing tokens.
39. **How to choose a shard key?** High cardinality, even distribution of data and traffic, matches the main query so most queries hit one shard.
40. **Range vs hash sharding?** Range: efficient range scans, hot spots on recent keys. Hash: even spread, range queries hit all shards.
41. **What is resharding and how do you make it painless?** Moving data when adding shards; use many logical partitions or consistent hashing so only a fraction moves.
42. **What is the celebrity / hot-key problem?** One key gets disproportionate traffic; mitigate with caching, key salting/splitting, dedicated shards.
43. **Local vs global secondary indexes?** Local: per shard, writes local, reads scatter-gather. Global: partitioned by indexed value, reads targeted, writes cross shards (async).

## Caching
44. **Cache-aside vs write-through vs write-behind?** Cache-aside: app fills cache on miss. Write-through: write cache+DB together. Write-behind: write cache, flush DB later (risk of loss).
45. **Why delete rather than update cache on write?** Avoids races where concurrent writers leave an older value in the cache.
46. **Cache stampede fix?** Single-flight/locks, early probabilistic refresh, serve stale while revalidating.
47. **Cache penetration fix?** Cache negative results and use a Bloom filter for non-existent keys.
48. **Cache avalanche fix?** TTL jitter, multi-level caches, warm-up, circuit breakers on the DB.
49. **LRU vs LFU?** LRU evicts least recently used (recency); LFU evicts least frequently used (popularity).
50. **Redis vs Memcached?** Redis: rich data types, persistence, replication. Memcached: simple multi-threaded string cache.

## Consistency & distributed systems
51. **Explain CAP.** During a network partition you must choose consistency or availability; partitions are unavoidable.
52. **What does PACELC add?** Even without partitions, there's a latency vs consistency trade-off.
53. **Strong vs eventual consistency examples?** Strong: payments, inventory, usernames. Eventual: likes, feeds, view counts.
54. **What is a quorum?** With N replicas, writes to W and reads from R; `W + R > N` makes reads see the latest write.
55. **Vector clocks vs last-write-wins?** Vector clocks detect concurrent writes; LWW silently picks one using timestamps.
56. **What are CRDTs?** Data types whose concurrent updates always merge deterministically (counters, sets, text) - used for multi-leader/offline sync.
57. **What is consensus and where is it used?** Agreement among nodes despite failures (Raft/Paxos) - leader election, config stores, replicated logs.
58. **How does Raft elect a leader?** Follower times out, becomes candidate, requests votes; majority wins; randomized timeouts avoid split votes.
59. **Why odd-sized clusters?** Majority quorum: 5 nodes tolerate 2 failures; 6 nodes still tolerate only 2.
60. **What is a fencing token?** Increasing number issued with each lock; storage rejects writes with older tokens - protects against paused ex-holders.
61. **Gossip protocol?** Nodes periodically share state with random peers; information spreads in O(log N) rounds; used for membership/failure detection.
62. **Lamport vs vector clocks?** Lamport gives a total order consistent with causality; vector clocks can also detect concurrency.
63. **Why not trust wall clocks for ordering?** Clocks drift and jump; use logical clocks, sequence numbers, or bounded-uncertainty clocks (TrueTime).

## Transactions & messaging
64. **Why avoid 2PC across microservices?** Blocking on coordinator failure, locks held across the network, availability depends on all participants.
65. **Saga choreography vs orchestration?** Choreography = services react to events (decoupled, hard to trace); orchestration = central coordinator (clear flow).
66. **Transactional outbox?** Store the event in the same DB transaction as the data; a relay publishes it - no dual-write inconsistency.
67. **At-least-once vs exactly-once?** At-least-once may duplicate; exactly-once = at-least-once + idempotent processing/dedupe (or broker transactions).
68. **Kafka vs RabbitMQ?** Kafka: durable replayable log, partition ordering, massive throughput. RabbitMQ: flexible routing, per-message acks, task queues.
69. **How is ordering guaranteed in Kafka?** Only within a partition; use the same key for related events.
70. **What is a dead letter queue?** Where messages go after repeated failures, so they don't block processing and can be inspected.
71. **What is backpressure?** Signals/limits that slow producers when consumers can't keep up (bounded queues, rejecting, scaling consumers).
72. **What is CDC?** Streaming database changes from its log (binlog/WAL) as events - feeds caches, search, warehouses, outbox.

## Reliability & operations
73. **SLI vs SLO vs SLA?** Indicator (measured), objective (internal target), agreement (contract with penalties).
74. **What is an error budget?** 100% − SLO; when spent, slow down risky changes.
75. **Why exponential backoff with jitter?** Avoids synchronized retry storms that overload recovering services.
76. **How does a circuit breaker work?** Closed -> open after failures (fail fast) -> half-open trial -> closed if healthy.
77. **What is a bulkhead?** Isolating resources per dependency/tenant so one failure can't exhaust everything.
78. **Canary vs blue-green deployment?** Canary: gradually shift traffic and watch metrics; blue-green: switch all traffic between two environments, instant rollback.
79. **RPO vs RTO?** RPO = acceptable data loss window; RTO = acceptable downtime.
80. **What is graceful degradation?** Serving reduced functionality (cached/default results) instead of failing completely.
81. **What is load shedding?** Rejecting low-priority requests early under overload to protect core functionality.

## Observability & security
82. **Golden signals?** Latency, traffic, errors, saturation.
83. **Why p99 and not average?** Averages hide tail latency that many users experience, especially with fan-out.
84. **Logs vs metrics vs traces?** Metrics = cheap aggregates for alerting; logs = detailed events; traces = request path across services.
85. **Head vs tail sampling of traces?** Head decides at start (cheap); tail decides after completion (keeps errors/slow traces).
86. **Session cookies vs JWT?** Sessions: server lookup, easy revocation. JWT: stateless verification, hard to revoke -> short expiry + refresh tokens.
87. **OAuth 2.0 vs OIDC?** OAuth = delegated authorization; OIDC adds identity (ID token) for login.
88. **How do you store passwords?** Salted slow hashes: bcrypt, scrypt, Argon2.
89. **What is IDOR?** Accessing others' resources by changing IDs; prevent with server-side ownership checks.
90. **How do you mitigate DDoS?** CDN/anycast absorption, WAF, rate limiting, autoscaling, challenge pages.
91. **What is envelope encryption?** Data encrypted with a data key; data key encrypted by a master key in KMS.

## Storage, data processing & data structures
92. **Block vs file vs object storage?** Block = raw disk volumes (DBs); file = shared hierarchical FS; object = HTTP-accessed immutable blobs at huge scale (media, backups).
93. **Replication vs erasure coding?** Replication: simple, 200% overhead; erasure coding: ~50% overhead, more CPU and slower repair.
94. **OLTP vs OLAP?** OLTP: many small transactions, row storage. OLAP: big analytical scans, columnar storage.
95. **Batch vs stream processing?** Batch: bounded data, high latency, simple correctness. Stream: unbounded, low latency, needs windows/watermarks.
96. **Event time vs processing time?** When it happened vs when processed; correct aggregations use event time with watermarks.
97. **Bloom filter?** Probabilistic set membership: no false negatives, some false positives, tiny memory.
98. **Count-Min Sketch?** Approximate frequency counts in fixed memory; used for heavy hitters.
99. **HyperLogLog?** Approximate distinct counts (~12 KB for billions, ~1% error), mergeable.
100. **Geohash vs quadtree?** Geohash: fixed grid strings, easy to store/query by prefix; quadtree: adaptive in-memory tree that adjusts to density.

---

**Next:** [Scenario & troubleshooting questions](02-scenario-questions.md) · [Estimation drills](03-estimation-drills.md) · [Quick design drills](04-quick-design-drills.md)
