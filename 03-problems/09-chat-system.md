# 09 · Design a Chat System (WhatsApp / Messenger / Slack)

> **Category:** Real-time messaging · **Difficulty:** ⭐⭐⭐⭐ · **Patterns:** WebSockets, stateful connection servers, session registry, per-channel ordering, sync by max message ID, presence via heartbeats

---

## 1. TL;DR
Clients keep a **WebSocket** to a stateful **chat server**. Messages get a per-conversation increasing **message ID**, are stored in a write-optimised **wide-column DB** (Cassandra/HBase partitioned by channel), then routed to the recipient's chat server (found through a **session registry** in Redis) or sent as a **push notification** if offline. Devices sync by asking for messages with ID > their last seen ID. **Presence** uses heartbeats.

---

## 2. Requirements
| Functional | Non-functional |
|---|---|
| 1:1 chat and groups (≤ 100 members) | Low latency delivery (< 500 ms) |
| Online presence | 50M DAU |
| Multi-device sync | Durable message history (forever) |
| Push notifications when offline | Message order per conversation |
| Text (media optional) | Highly available |

---

## 3. Connection protocol choice
| Option | Fit |
|---|---|
| Short polling | ❌ wasteful |
| Long polling | ⚠️ works, but sender/receiver may hit different servers, hard to detect disconnects |
| **WebSocket** | ✅ bidirectional, persistent, starts as HTTP upgrade (passes firewalls) |

---

## 4. Architecture

```mermaid
flowchart TB
    subgraph Stateless["Stateless services (HTTP)"]
      API["API: login, profile, groups, history"]
      SD["Service discovery: pick best chat server"]
    end
    subgraph Stateful["Stateful (WebSocket)"]
      CS1["Chat server 1"]
      CS2["Chat server 2"]
    end
    C1["Client A"] -->|"WebSocket"| CS1
    C2["Client B"] -->|"WebSocket"| CS2
    C1 --> API
    C1 --> SD
    CS1 --> IDG["Message ID generator"]
    CS1 --> MQ["Message sync queue / pub-sub"]
    MQ --> CS2
    CS1 --> KV[("Message store: Cassandra")]
    CS1 --> REG[("Session registry: user to server (Redis)")]
    CS1 --> PR["Presence servers"]
    MQ --> PN["Push notification service"]
```

---

## 5. 1:1 message flow
```mermaid
sequenceDiagram
    participant A as User A
    participant S1 as Chat Server 1
    participant DB as Message Store
    participant R as Session Registry
    participant S2 as Chat Server 2
    participant B as User B
    A->>S1: send(msg, client_msg_id)
    S1->>S1: assign message_id (per channel)
    S1->>DB: persist
    S1-->>A: ack (sent)
    S1->>R: where is B?
    alt B online
        R-->>S1: server 2
        S1->>S2: forward
        S2->>B: deliver
        B-->>S2: delivered ack
    else B offline
        S1->>S1: send push notification
    end
```

---

## 6. Data model
```
messages (
  channel_id   bigint,   -- partition key (1:1 = sorted pair of user IDs)
  message_id   bigint,   -- clustering key, descending
  sender_id    bigint,
  content      text,
  created_at   timestamp,
  PRIMARY KEY (channel_id, message_id))
```
Why wide-column? Huge write volume, read pattern is "latest N messages of a channel", linear scaling. Very large channels → partition by `(channel_id, time_bucket)`.

**Message IDs:** must be ordered within a channel. Options: Snowflake (roughly ordered), per-channel sequence counter (Redis `INCR`), or a local sequence on the server owning the channel.

---

## 7. Deep dives
- **Multi-device sync:** each device stores `cur_max_message_id`; on connect it fetches newer messages.
- **Group chat:** small groups → copy message reference into each member's sync queue (fan-out on write); large channels (Slack/Discord) → store once, members pull on open, push only to online members.
- **Delivery states:** sent ✓ (stored), delivered ✓✓ (device ack), read (blue) → track as "last read message_id" per user per channel.
- **Idempotency:** client-generated `client_msg_id` dedupes resends.
- **Presence:**
```mermaid
flowchart LR
    HB["Client heartbeat every 5s"] --> P["Presence server"]
    P --> KV[("status: user to online/last_seen")]
    P -->|"no heartbeat for 30s"| OFF["mark offline"]
    P --> PS["publish status change to friends' channels"]
```
Heartbeats avoid flicker when the network briefly drops. For huge friend lists, fetch presence lazily instead of broadcasting.
- **Chat server failure:** clients reconnect to another server via service discovery and resync by message ID.
- **E2E encryption:** Signal protocol — server only relays ciphertext.
- **Media:** upload to object storage via pre-signed URL; message carries the URL/thumbnail.

---

## 8. Capacity sketch
50M DAU × 40 msgs = 2B msgs/day ≈ 23K/s (peak ~70K). 10M concurrent connections ÷ ~100K per server ≈ 100 chat servers.

---

## 9. Interview Q&A
**Q: Why separate stateless and stateful services?** Stateful WebSocket servers are hard to scale/deploy; keep them thin (only connection & routing) and do everything else in stateless services.

**Q: How do you guarantee order?** Per-channel monotonically increasing IDs assigned by one place; clients sort by ID, not by device time.

**Q: How do you load-balance WebSockets?** Service discovery (ZooKeeper/etcd) picks the least-loaded server by region; connections are long-lived so use least-connections.

**Q: Message delivered twice after reconnect?** Dedupe by message_id on the client.

---

**Related:** [Networking: real-time options](../01-fundamentals/01-networking-basics.md) · [Slack](58-slack.md) · [Google Docs](35-google-docs.md) · [Notifications](07-notification-system.md)
