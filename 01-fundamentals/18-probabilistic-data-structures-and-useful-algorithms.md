# Probabilistic Data Structures & Algorithms Every Designer Should Know

> These are the "secret weapons" that make designs at billion-scale feasible. Name-dropping them correctly (with *why*) earns big points.

---

## 1. Bloom Filter — "Is X definitely NOT in the set?"
- A bit array of `m` bits + `k` hash functions.
- **Add:** set bits `h1(x)..hk(x)` to 1.
- **Query:** if any bit is 0 → **definitely not present**; if all 1 → **probably present** (false positives possible, **no false negatives**).
- Memory: ~10 bits per element for 1% false positive rate (vs storing the whole key).
- Can't delete (use **Counting Bloom filter** or **Cuckoo filter**).

**Used in:** web crawler "URL seen?", Cassandra/HBase/RocksDB (skip SSTables that don't contain a key), cache penetration protection, Medium "already read" articles, Chrome malicious URL checks, username availability pre-checks.

---

## 2. Count-Min Sketch — approximate frequencies
- 2D array `d` rows × `w` columns, one hash function per row.
- **Increment(x):** for each row `i`, `table[i][h_i(x)] += 1`.
- **Estimate(x):** `min_i table[i][h_i(x)]` (overestimates only, never under).
- Fixed memory regardless of number of distinct items.

**Used in:** top-K / heavy hitters (trending hashtags, most-viewed videos), rate limiting by many keys, network traffic monitoring.
Combine with a **min-heap of size K** to track the top K.

---

## 3. HyperLogLog — approximate distinct count (cardinality)
- Hash each item; track the max number of leading zeros seen in each of `m` registers; combine with harmonic mean.
- ~**12 KB** to count billions of distinct items with ~0.8% error.
- **Mergeable**: union of HLLs = HLL of union → per-hour HLLs can be merged to daily.

**Used in:** unique visitors (Redis `PFADD`/`PFCOUNT`), distinct users per ad, BigQuery `APPROX_COUNT_DISTINCT`.

---

## 4. Merkle Tree — efficient diff of large datasets
- Leaves = hashes of data blocks; each parent = hash of children; root = fingerprint of everything.
- Two replicas compare roots; if different, recurse only into differing subtrees → find differences in O(log n) comparisons.

**Used in:** Cassandra/Dynamo anti-entropy repair ([Distributed Key-Value Store (Dynamo / Cassandra style)](../03-problems/03-key-value-store.md)), Git, blockchains, certificate transparency, file sync (Dropbox), ZFS.

---

## 5. Consistent Hashing — see [Consistent Hashing](../03-problems/02-consistent-hashing.md)
Ring of hash space with virtual nodes; adding/removing a node moves only ~1/N of keys. Variants: **Rendezvous (HRW) hashing**, **Jump consistent hash**, **Maglev hashing** (Google LB).

---

## 6. Geospatial indexing
| Technique | Idea | Used by |
|---|---|---|
| **Geohash** | Interleave lat/long bits → base32 string; shared prefix ≈ nearby | Redis GEO, Elasticsearch, Yelp-style search |
| **Quadtree** | Recursively split 2D space into 4 until each cell has ≤ N points (adapts to density) | Uber (early), in-memory location search |
| **S2 cells (Google)** | Project sphere onto cube faces, Hilbert curve ordering → 64-bit cell ids | Google Maps, Foursquare, Uber (H3 is hexagonal variant) |
| **H3 (Uber)** | Hexagonal hierarchical grid; equal-distance neighbors | Uber surge pricing, dispatch |
| **R-tree** | Bounding rectangle tree | PostGIS, spatial DBs |

([Proximity Service (Yelp "restaurants near me")](../03-problems/13-proximity-service.md)) – Proximity Service explains geohash and quadtree in depth.)

---

## 7. Trie (prefix tree)
Each node = a character; paths = prefixes. Store top-K completions at each node for O(prefix length) autocomplete ([Search Autocomplete (Typeahead)](../03-problems/10-search-autocomplete.md)).

---

## 8. Skip list
Probabilistic layered linked list with O(log n) search/insert; simpler than balanced trees and lock-friendly. **Redis sorted sets** use skip lists (leaderboards, [Real-Time Gaming Leaderboard](../03-problems/22-gaming-leaderboard.md). Also used in LSM memtables.

---

## 9. Inverted index
`term → posting list of (doc_id, positions, frequency)`. The core of full-text search (see Search Systems note).

---

## 10. Rate-limiting algorithms
Token bucket, leaky bucket, fixed window, sliding window log, sliding window counter ([Rate Limiter](../03-problems/01-rate-limiter.md)).

---

## 11. Other handy algorithms
- **Reservoir sampling** – keep a uniform random sample of k items from a stream of unknown length.
- **Lamport / Vector clocks** – ordering events (see Clocks note).
- **Gossip protocol** – epidemic dissemination of membership/state ([Distributed Key-Value Store (Dynamo / Cassandra style)](../03-problems/03-key-value-store.md)).
- **Operational Transformation (OT) / CRDTs** – real-time collaborative editing (see Google Docs problem).
- **Snowflake IDs** – time-sortable unique IDs ([Unique ID Generator for Distributed Systems](../03-problems/04-unique-id-generator.md)).
- **Base62 encoding** – short URLs ([URL Shortener (TinyURL / bit.ly)](../03-problems/05-url-shortener.md)).
- **SimHash / MinHash** – near-duplicate detection (web crawler, plagiarism).
- **Hashed timing wheel** – efficient timers for millions of timeouts (Kafka, Netty).
- **Rolling hash (Rabin fingerprint)** – content-defined chunking for dedupe (Dropbox, backups).

---

## 12. Cheat table

| Problem | Structure |
|---|---|
| "Have we seen this URL/user before?" | Bloom filter |
| "How many unique users today?" | HyperLogLog |
| "Top 10 trending hashtags?" | Count-Min Sketch + heap |
| "Which replica data differs?" | Merkle tree |
| "Restaurants within 2 km?" | Geohash / Quadtree / S2 / H3 |
| "Autocomplete for prefix?" | Trie with cached top-K |
| "Real-time rank among 10M players?" | Redis sorted set (skip list) |
| "Distribute keys across changing servers?" | Consistent hashing |
| "Near-duplicate web pages?" | SimHash |
