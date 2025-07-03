# 🧠 Redis System Design

This repository contains a comprehensive summary of Redis concepts, patterns, and interview-style questions to help you prepare for system design interviews or deepen your practical understanding of Redis.





## 📚 Topics Covered

### 🔹 Redis Basics
- Written in C, in-memory, single-threaded (very fast)
- Core data model: key-value store with rich data types
- Simple CLI & wire protocol
- Persistence options: AOF, RDB (optional, not always durable)

---

### 🧱 Data Structures
- Strings, Lists, Hashes, Sets, Sorted Sets
- Geospatial indexes
- Bitmaps, HyperLogLogs, Bloom filters
- Streams (durable message logs)

---

### ⚙️ Infrastructure Configurations
- Single Node
- Master-Replica (High Availability)
- Cluster Mode (uses 16,384 hash slots)

---

### ⚡ Use Cases & Patterns

#### 1. **Caching (Cache-Aside, Write-Through, TTL)**
- TTL: auto-expiry with `EXPIRE`
- Eviction strategies: LRU, LFU, Random
- Hot key mitigation techniques

#### 2. **Distributed Locking**
- Basic: `SET key uuid NX PX 3000`
- Safe release with Lua: `GET` + `DEL`
- Advanced: Redlock algorithm (quorum-based locking)

#### 3. **Leaderboards**
- Use Sorted Sets (`ZADD`, `ZREVRANGE`)
- Rank users/posts by score

#### 4. **Rate Limiting**
- Fixed-window limiter: `INCR` + `EXPIRE`
- Limitations: burstiness, hot keys
- Alternatives: sliding window, token bucket (Lua scripts)

#### 5. **Pub/Sub vs Streams**
| Feature             | Pub/Sub                | Streams                       |
|---------------------|------------------------|-------------------------------|
| Persistent?         | ❌ No                  | ✅ Yes                        |
| Replay messages?    | ❌ No                  | ✅ Yes                        |
| Delivery guarantee  | At most once           | At least once (with ack)     |
| Best for            | Real-time (chat, notif)| Durable work queues          |

#### 6. **Work Queues (Streams)**
- `XADD`, `XREADGROUP`, `XACK`, `XCLAIM`
- Consumer groups for parallel processing
- Retry unacknowledged tasks

#### 7. **Proximity Search**
- `GEOADD`, `GEOSEARCH` for geospatial queries

---

### 🧠 Redis Cluster & Hash Slots

- Cluster uses 16,384 hash slots to distribute keys
- Keys are routed to slots via `CRC16(key) % 16384`
- Related keys grouped using `{hash_tag}` for same-slot ops
- Multi-key commands require same slot → design keys carefully

---

### 🧪 Common Interview Questions (With Topics)

1. What makes Redis fast? *(in-memory, single-threaded, efficient C code)*
2. Trade-offs of in-memory design
3. Why single-threaded Redis still handles high throughput
4. When to use Sorted Sets vs Lists
5. Modeling a shopping cart in Redis
6. Set vs Sorted Set differences
7. How does Redis handle cache eviction?
8. How to prevent stale cache data
9. What is a hot key and how to fix it
10. How to implement distributed locks (SET NX PX + UUID + Lua)
11. What is the Redlock algorithm?
12. Failure scenarios in Redis-based locking
13. Pub/Sub vs Streams
14. Durable work queue with Streams
15. What happens if a consumer fails (use XPENDING/XCLAIM)
16. How Redis Cluster distributes data (hash slots)
17. Importance of hash slots
18. Redis multi-key command limits
19. Fixed-window rate limiter in Redis
20. Limitations of Redis for rate limiting

---

### 📦 Key Redis Commands

```bash
SET key value NX PX 3000          # Acquire lock
GET key                           # Read key
DEL key                           # Release lock
EXPIRE key 60                     # TTL for cache or rate limit
INCR key                          # Rate limiting counter
XADD mystream * key value         # Add to stream
XREADGROUP GROUP grp consumer ...# Read from stream
XACK mystream grp message_id     # Acknowledge stream message
SPUBLISH channel msg              # Pub/Sub
SSUBSCRIBE channel                # Pub/Sub
GEOADD key lon lat member         # Geospatial indexing
GEOSEARCH key FROMLONLAT lon lat BYRADIUS r km
