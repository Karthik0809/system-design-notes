# Back-of-the-Envelope Estimation Drills (with worked solutions)

Use: 1 day ≈ 10⁵ s · 1M/day ≈ 12/s · peak ≈ 2-3× average · replication ×3 · round aggressively.

---

### Drill 1 - Twitter-scale writes and storage
300M DAU, 2 tweets/user/day, 10% with 1 MB media, keep 5 years.
- Tweets/day = 600M -> **~7K writes/s**, peak ~15K.
- Text: 600M × 300 B = 180 GB/day.
- Media: 60M × 1 MB = **60 TB/day** -> 5 yrs ≈ 110 PB (×3 replicas or erasure coding).
- Conclusion: media must live in object storage + CDN; text in a sharded store.

### Drill 2 - Chat connections
50M DAU, 30% online at peak = 15M concurrent WebSockets; one server holds ~100K -> **150 chat servers** (+ headroom -> ~200).

### Drill 3 - Image serving bandwidth
20K image views/s × 200 KB = 4 GB/s = **32 Gbps** -> CDN required; origin sees only cache misses (~5% -> 1.6 Gbps).

### Drill 4 - Cache size for URL shortener
100M redirects/day, 20% of URLs get 80% of traffic, ~20M hot URLs × 500 B = **10 GB** -> one Redis node (+ replica).

### Drill 5 - Ride-hailing location updates
2M active drivers, update every 4 s -> **500K writes/s**; each update 100 B -> 50 MB/s; keep only latest in memory: 2M × 100 B = **200 MB** (tiny) - the challenge is write rate, not size.

### Drill 6 - Metrics system
5,000 hosts × 500 metrics × 1 sample / 10 s = **250K points/s**; at 2 bytes/point compressed -> 0.5 MB/s -> ~43 GB/day raw.

### Drill 7 - Video platform storage
500K uploads/day × 500 MB avg (original + renditions) = **250 TB/day** -> ~90 PB/year -> tiering & deleting unpopular renditions matter.

### Drill 8 - Number of app servers
Peak 120K QPS; each server handles 2K QPS at 50% CPU target -> 60 servers; N+2 redundancy across 3 AZs -> **~66-75**.

### Drill 9 - Database sizing for orders
10M orders/day × 2 KB (row + indexes) = 20 GB/day -> 7 TB/year -> shard after ~1-2 years or archive old orders to cold storage/warehouse.

### Drill 10 - Notification campaign
Send 50M push notifications within 30 minutes -> 50M / 1,800 s ≈ **28K/s**; if one worker does 500/s -> ~56 workers; check provider rate limits.

### Drill 11 - Search index size
1B documents × 1 KB of text -> 1 TB raw; inverted index ≈ 30-100% of raw -> ~0.5-1 TB; with 2 replicas ~3 TB; at 50 GB/shard -> ~20 primary shards.

### Drill 12 - LLM serving capacity
2K requests/s × 400 output tokens = 800K tokens/s; one GPU replica ~2.5K tokens/s with batching -> **~320 replicas** (+ headroom for peaks and prefill).

---

## Estimation checklist
1. Users (DAU/MAU) -> actions per user -> requests/s (avg & peak).
2. Read:write ratio.
3. Object sizes -> storage per day/year (× replication).
4. Bandwidth in/out.
5. Memory for caches (hot set).
6. Servers needed (QPS ÷ per-server capacity).
7. **State the conclusion** ("needs sharding", "fits in memory", "needs CDN").

**See also:** [Back-of-the-envelope estimation](../01-fundamentals/22-back-of-the-envelope-estimation.md)
