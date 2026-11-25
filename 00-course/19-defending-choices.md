# Chapter 19 · "What Does That Choice Cost You?"

## The situation
Onsite interview, minute 30. You've drawn Crumb's comment shards on Cassandra. The interviewer points at it: "Why not PostgreSQL?" You say "Cassandra scales better." They ask: "What does that cost you?" Silence.

## Why it goes wrong
"X scales better" is a slogan, not a reason. Interviewers want three things for every major choice:
1. **The requirement** that drove it.
2. **The cost** you're accepting.
3. **What would change your mind.**

## A template you can say out loud
> "Because [requirement], I chose [option]. The cost is [downside], which is fine here because [reason]. If [condition] changed, I'd switch to [alternative]."

**Applied to the comment store:**
> "Comments are written 8,000 times a second, read almost always as 'comments for one recipe, newest first', and we have 6 billion of them growing 30 GB a day. Cassandra partitioned by recipe ID fits that access pattern and scales writes by adding nodes. The cost: no joins, no multi-row transactions, and only the queries we design tables for; 'all comments by a user' needs a second table. That's fine because comments are simple, independent rows. If we needed rich ad-hoc queries or transactions across comments, I'd keep PostgreSQL and shard it with Citus or Vitess."

## Crumb's decisions, each with its price
| Decision | Requirement behind it | Cost accepted | Would change if |
|---|---|---|---|
| Cache-aside with TTL | Reads 3,000x writes | Seconds of staleness | Data must be exact (prices at checkout) |
| Async read replicas | Read capacity | Stale reads, read-your-writes logic | Strong reads everywhere required |
| Hybrid feed fan-out | Fast feed opens | Write amplification, complexity | Mostly inactive users or tiny follow graphs |
| Queue for photo resizing | Fast publish | Thumbnails appear seconds later | Users need final images instantly |
| Snowflake IDs | Unique IDs without a central DB | Clock dependence | Single DB is enough forever |
| Home-region writes | Low latency, region failover | Async loss window, conflicts | Money or uniqueness involved |
| Eventual consistency for likes | Availability, cheap counters | Counts briefly off | Billing based on counts |

## CAP, said plainly
When the network between replicas breaks, a replicated system must choose: **refuse some requests** (consistent) or **answer with possibly stale data** (available). Say which you choose **per feature**: likes and feeds stay available; usernames and payments stay consistent.

## What strong answers sound like
- They use the numbers from the estimate.
- They admit the downside before being asked.
- They prefer the simpler option until a requirement forces the complex one.
- They don't flip the design at the first challenge; they reason about it ("If that's a requirement, here's what changes").

## Try it
The interviewer asks: "Why not just put everything in Redis? It's fast." Answer with the template.

<details><summary>Answer</summary>

"Redis is a great cache because our reads are hot and repetitive, but as the only store it costs us durability (memory first, async persistence can lose recent writes), capacity (all data in RAM is expensive at 14 TB) and query flexibility. So I keep PostgreSQL/Cassandra as the source of truth and Redis as a cache. If the data were small, ephemeral and loss-tolerant, like sessions or rate-limit counters, Redis alone would be fine."
</details>

**Next:** [Chapter 20: Run the clock](20-running-the-clock.md) · **Deeper:** [Making trade-offs](../01-fundamentals/32-making-trade-offs.md), [CAP and consistency](../01-fundamentals/08-cap-pacelc-and-consistency-models.md)
