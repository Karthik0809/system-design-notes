# System Design Interview Cheat Sheet & Problem Patterns

Use this page for quick revision before an interview. Every link points to a deeper note in this repo.

---

## 1. The 45-minute game plan (see [The System Interview Framework](23-interview-framework.md)
| Time | Step | Output |
|---|---|---|
| 0-5 min | **Requirements** | 3-5 functional features, non-functional (scale, latency, consistency, availability) |
| 5-10 min | **Estimates** | QPS (avg & peak), storage/year, bandwidth, read:write ratio |
| 10-15 min | **API + data model** | 3-5 endpoints; main tables/entities & choice of DB |
| 15-25 min | **High-level diagram** | Clients -> LB -> services -> cache/DB/queue/blob/CDN |
| 25-40 min | **Deep dives** | 2-3 hardest parts (the interviewer usually picks) |
| 40-45 min | **Wrap-up** | Bottlenecks, failure modes, monitoring, future scale |

---

## 2. Numbers to remember
- 1 day ≈ **86,400 s ≈ 10⁵ s**. 1M requests/day ≈ **12 req/s**. 100M/day ≈ 1.2K/s.
- Peak ≈ 2-10× average.
- 1 char ≈ 1 byte (ASCII); UUID 16 B; Snowflake ID 8 B; tweet ≈ 300 B; photo ≈ 200 KB-2 MB; 1 min HD video ≈ 50-100 MB.
- Single Postgres/MySQL: ~10K simple QPS (reads more with cache); Redis: ~100K ops/s per node; Kafka: ~1M msgs/s per cluster (easily).
- A server: ~10-50K concurrent WebSocket connections (well-tuned: 100K-1M).
- Memory read 100 ns; SSD read 100 µs; datacenter RTT 0.5 ms; cross-continent RTT ~150 ms.
- Availability: 99.9% = 8.8 h/yr down; 99.99% = 52 min/yr.

---

## 3. Requirement -> Component mapping

| If you need... | Reach for... | Note |
|---|---|---|
| Scale stateless compute | LB + autoscaled stateless servers | [Load Balancing](03-load-balancing.md) |
| Fast reads of hot data | Redis cache (cache-aside) | [Caching](07-caching-deep-dive.md) |
| Static/media delivery | Object storage + CDN | [Storage](16-storage-systems-block-file-object-oltp-vs-olap.md) |
| Strong consistency / transactions | SQL (Postgres) or Spanner/CockroachDB | [Databases](04-databases-sql-vs-nosql-acid-indexing.md) |
| Massive write throughput | Cassandra / DynamoDB (LSM) | [Databases](04-databases-sql-vs-nosql-acid-indexing.md) |
| More data than one DB | Sharding by a good key | [Sharding](06-partitioning-and-sharding.md) |
| High availability of data | Replication + failover | [Replication](05-replication.md) |
| Async work / decoupling / spikes | Kafka / SQS | [Queues](11-message-queues-pub-sub-and-event-streaming.md) |
| Real-time server -> client | WebSocket / SSE | [Networking](01-networking-basics.md) |
| Full-text search | Elasticsearch via CDC | [Search](19-search-systems-and-inverted-index.md) |
| Nearby things | Geohash / Quadtree / H3 | [Proximity Service (Yelp "restaurants near me")](../03-problems/13-proximity-service.md), [Google Maps (Rendering, Routing, ETA)](../03-problems/15-google-maps.md), [Uber (Ride Hailing)](../03-problems/28-uber.md) |
| Unique IDs | Snowflake | [Unique ID Generator for Distributed Systems](../03-problems/04-unique-id-generator.md) |
| Leader election / locks | etcd / ZooKeeper + fencing | [Consensus](09-consensus-leader-election-and-distributed-locks.md) |
| Multi-service transaction | Saga + Outbox | [Transactions](10-distributed-transactions-2pc-saga-outbox.md) |
| Analytics / aggregation | Kafka -> Flink -> OLAP; Spark batch | [Batch & Stream](17-batch-and-stream-processing.md) |
| Dedupe / membership at scale | Bloom filter | [Probabilistic DS](18-probabilistic-data-structures-and-useful-algorithms.md) |
| Unique counts | HyperLogLog | same |
| Top-K | Count-Min Sketch + heap | same, [Top-K Heavy Hitters (Trending)](../03-problems/34-top-k-heavy-hitters.md) |
| Protect from abuse | Rate limiter | [Rate Limiter](../03-problems/01-rate-limiter.md) |

---

## 4. Problem patterns (recognise the family!)

| Pattern | Core trick | Problems (number in 03-problems) |
|---|---|---|
| **Read-heavy key lookup** | Cache + CDN + replicas, unique key generation | URL Shortener (5), Pastebin (33) |
| **Fan-out feed** | Push vs pull vs hybrid, celebrity problem | News Feed (8), Twitter (26), Instagram (27) |
| **Real-time messaging** | WebSockets, presence, per-conversation ordering | Chat (9), Google Docs (35), Video conf (39), Live chat (40) |
| **Media pipeline** | Pre-signed upload -> async transcoding (DAG) -> CDN | YouTube (11), Instagram (27), Netflix (46), Live Streaming (40), Spotify (48), Google Photos (67) |
| **File sync** | Chunking, dedupe, metadata DB, notifications | Google Drive (12) |
| **Geo / location** | Spatial index, frequent location updates, matching | Proximity (13), Nearby Friends (14), Maps (15), Uber (28), Food Delivery (30) |
| **Inventory / booking contention** | Conditional updates, holds with TTL, idempotency, waiting room | Hotel (19), Ticketmaster (29), Flash Sale (38) |
| **Money movement** | Idempotency keys, double-entry ledger, reconciliation, sagas, event sourcing | Payments (23), Wallet (24), Stock Exchange (25) |
| **Counting & aggregation** | Stream windows, pre-aggregation, sketches, lambda architecture | Ad Clicks (18), Top-K (34), Counters (42), Metrics (17), Web analytics (68) |
| **Ranking / scoring** | Sorted sets, two-stage retrieval | Leaderboard (22), Search (36), Recommendations (43), Autocomplete (10) |
| **Distributed infrastructure** | Partitioning, replication, consensus, gossip | KV Store (3), Cache (31), Message Queue (16), S3 (21), Lock Service (44), Logging (41) |
| **Job/task execution** | Queue + workers, retries, idempotency, scheduling | Notifications (7), Web Crawler (6), Job Scheduler (32), Code Judge (37), Webhooks (47), Email (20) |
| **GPU-bound serving** | Batching, KV cache, admission control | LLM Chat (45) |
| **Graphs and matching** | Sharded adjacency lists, set intersections, precomputed candidates | LinkedIn graph (64), Tinder (65) |
| **Developer and edge infrastructure** | Stateless proxies, config push, progressive rollout, metadata master | API gateway (63), CI/CD (62), Distributed file system (61), CDN (52), DNS (59) |
| **Real-time games** | Matchmaking queues, authoritative servers, UDP ticks | Multiplayer game backend (66) |

---

## 5. Deep-dive checklist (things interviewers love)
- [ ] Single points of failure removed (LB pair, DB replicas, multi-AZ).
- [ ] Hot keys / celebrity problem addressed.
- [ ] Idempotency for retried writes.
- [ ] Consistency level chosen per feature (and justified).
- [ ] Cache invalidation & stampede protection.
- [ ] Sharding key & rebalancing.
- [ ] Async processing for slow/side-effect work.
- [ ] Rate limiting & abuse.
- [ ] Monitoring: golden signals, alerts on SLOs.
- [ ] Failure scenarios: node, AZ, region, dependency down.

---

## 6. Common trade-offs to articulate
- Consistency vs availability/latency (CAP/PACELC).
- Push vs pull (write amplification vs read latency).
- Normalisation vs denormalisation.
- SQL vs NoSQL.
- Sync vs async processing.
- Exact vs approximate (sketches).
- Batch vs stream.
- Strong vs eventual ordering.
- Build vs buy (managed services).
- Cost vs performance (cache sizes, replicas, storage tiers).
