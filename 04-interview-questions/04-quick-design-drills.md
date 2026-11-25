# Quick Design Drills — 25 Short Problems with 5-Minute Solutions

> For practising the "core idea" of a problem fast. Each answer = key components + the one trick that makes it work. Full write-ups exist for many in [03-problems](../03-problems/).

---

### 1. Design a URL shortener
Base62 of unique ID (or KGS random keys) → KV store `key → url` → Redis cache + CDN → 302 for analytics. → [Full](../03-problems/05-url-shortener.md)

### 2. Design a rate limiter for an API
Gateway middleware, token bucket per (user, endpoint) in Redis via Lua, 429 + Retry-After, fail open if Redis down. → [Full](../03-problems/01-rate-limiter.md)

### 3. Design a "who's online" indicator
Clients heartbeat every 30 s → Redis `SET online:{user} 1 EX 60`; friends' status fetched on demand or pushed via pub/sub when it changes.

### 4. Design a typeahead
Trie with top-K per node built offline from query logs; trending overlay from stream counts; debounce + browser cache. → [Full](../03-problems/10-search-autocomplete.md)

### 5. Design a parking lot reservation app
Spots per lot per time slot; reserve with conditional update `status=FREE → HELD` + TTL; payment; QR at gate. Same pattern as hotel booking.

### 6. Design a unique visitor counter for a website
HyperLogLog per page per day in Redis (`PFADD`/`PFCOUNT`), merge days for weekly; exact counts in batch if needed.

### 7. Design a "recently viewed items" list
Redis list per user `LPUSH` + `LTRIM 50`; TTL 30 days; persist async if needed.

### 8. Design a voting system for an election (one person, one vote)
Authenticate voter, `INSERT INTO votes(voter_id PK, choice)` → uniqueness enforced by DB; tallies via async aggregation; audit log; strong consistency on the vote write.

### 9. Design a coupon / promo code system
Codes table with usage limits; redeem via atomic `UPDATE … SET used = used + 1 WHERE code = ? AND used < max`; per-user usage table with unique (user, code).

### 10. Design an image thumbnail service
Upload to S3 → event → workers resize to fixed sizes (or on-the-fly resizing at the CDN edge with caching) → serve via CDN.

### 11. Design a key-value config store
Strongly consistent small store (etcd/Raft), watch API for change notifications, versioned values, clients cache locally. → [Lock service](../03-problems/44-distributed-lock-service.md)

### 12. Design a distributed cron
Jobs table with `next_run_at`, partitioned schedulers with leases, queue + workers, idempotent jobs. → [Full](../03-problems/32-distributed-job-scheduler.md)

### 13. Design a news aggregator (Google News)
Crawl feeds → dedupe & cluster similar articles (SimHash/embeddings) → rank by freshness, source quality, personalization → cache per topic/region.

### 14. Design a comments system for a blog platform
Comments table partitioned by post_id, parent_id for threads, cursor pagination, spam filter on write, counts cached, moderation queue. → [Reddit](../03-problems/49-reddit.md)

### 15. Design a file sharing link with expiry
Random token → `share(token, file_id, expires_at, password_hash)`; download via pre-signed S3 URL generated after checks.

### 16. Design an inventory system for a warehouse
Per (sku, location) quantities, reservations with TTL for orders, event log of movements (ledger-like), reconciliation with physical counts.

### 17. Design a real-time multiplayer game backend
Matchmaking service (queue by skill/region) → assigns a dedicated game server (authoritative state, UDP ticks at 20–60 Hz) → results to leaderboard; region-local servers for latency.

### 18. Design a ride-sharing ETA service
Road graph tiles + contraction hierarchies, live traffic speeds per segment from GPS streams, ML correction, cache popular routes. → [Google Maps](../03-problems/15-google-maps.md)

### 19. Design a shopping cart
Cart per user in DynamoDB/Redis, merge guest cart at login, re-validate price/stock at checkout, cart isn't a reservation. → [E-commerce](../03-problems/38-ecommerce-flash-sale.md)

### 20. Design an email verification / OTP flow
Generate random code, store hashed with TTL (5 min) and attempt counter, rate-limit sends and verifies per user/IP, single-use.

### 21. Design a web analytics tracker (Google Analytics)
JS beacon → edge collectors → Kafka → stream aggregation (pageviews, sessions with session windows) → columnar store (ClickHouse/Druid) → dashboards; HLL for uniques.

### 22. Design a stock ticker display
Exchange feed → normalizer → Kafka → pub/sub fan-out per symbol → clients via WebSocket with throttling (e.g., max 4 updates/s/symbol); latest price cache for initial load. → [Price alerts](../03-problems/54-price-alerts.md)

### 23. Design an API for idempotent money transfer
`POST /transfers` with `Idempotency-Key`; store key + result; ledger double-entry in one transaction; return stored result on retry. → [Payments](../03-problems/23-payment-system.md)

### 24. Design a distributed counter for API usage billing
Gateway emits usage events → Kafka → aggregate per customer per hour (exactly-once) → billing DB; Redis counters for real-time quota checks. → [Counter](../03-problems/42-distributed-counter.md)

### 25. Design a collaborative whiteboard
WebSocket rooms, shapes as CRDT objects (per-shape LWW registers), op broadcast via room server, snapshots + op log, presence cursors. → [Google Docs](../03-problems/35-google-docs.md)

---

## How to practise
1. Set a 5-minute timer; write requirements, the main components, and the "one trick".
2. Then compare with the answer; note which **pattern** you missed (see [Patterns](../02-patterns/README.md)).
3. Pick the full write-up and do a 45-minute mock for the hardest ones.
