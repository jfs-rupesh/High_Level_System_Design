
# 📘 Redis Interview Questions & In-Depth Answers

---

## 1. What makes Redis so fast?
- Redis stores all data in RAM, making reads/writes extremely fast.
- It uses a single-threaded event loop (no locks, no context switching).
- Efficient data structures written in C.
- RESP protocol is lightweight and fast to parse.

---

## 2. What are the trade-offs of using Redis' in-memory architecture?
- Risk of data loss if Redis crashes (unless AOF is used).
- RAM is expensive; limits data size.
- In-memory means volatile unless configured for persistence.
- Great for performance, but less suitable for long-term storage.

---

## 3. Redis is single-threaded. How does it still support high throughput?
- Uses a non-blocking event loop.
- No locking overhead = more efficient CPU use.
- All commands are short and optimized (many O(1)/O(logN)).
- I/O multiplexing allows many concurrent connections.

---

## 4. When would you use a Sorted Set over a List?
- When you need ordering by score (e.g. leaderboard, top-N queries).
- When you need to update rankings.
- Sorted Sets ensure uniqueness + order; Lists allow duplicates and preserve insertion order.

---

## 5. How would you model a shopping cart using Redis?
- Hash: cart:{user_id} to store product_id -> quantity.
- Set: favorites:{user_id}.
- List: recent views with TTL.
- Sorted Set: trending product scores.
- Use EXPIRE on cart to auto-remove stale carts.
- Distributed locks for checkout consistency.

---

## 6. What’s the difference between a Set and a Sorted Set?
- Set: unordered, unique values.
- Sorted Set: ordered by score, unique members.
- Use Sorted Set when ranking is needed.

---

## 7. How does Redis handle cache eviction?
- Uses TTL (`EXPIRE`) to remove keys after a time.
- Eviction policies (e.g. LRU, LFU, random) if `maxmemory` is set.
- `DEL` and `UNLINK` support manual removal.

---

## 8. What strategies can you use to prevent stale data in a Redis cache?
- Set TTLs.
- Invalidate cache on DB writes.
- Write-through caching.
- Read-through caching.
- Versioning or timestamps.
- Use pub/sub to sync updates.

---

## 9. What is a “hot key” and how can you mitigate it?
- A key with high access volume causing node overload.
- Mitigations:
  - Use key sharding.
  - Client-side caching.
  - Add read replicas.
  - Load-balance with random key variants.

---

## 10. How would you implement a distributed lock using Redis?
- Use `SET lock:key uuid NX PX 3000`.
- Check response is OK before proceeding.
- Release with Lua: check UUID then `DEL`.
- Guarantees atomicity and auto-expiry.

---

## 11. What is the Redlock algorithm and why might you need it?
- Distributed lock across 5 Redis nodes.
- Acquire lock on majority (e.g. 3/5).
- Prevents split-brain and handles node failure.
- Used when high availability is needed.

---

## 12. What are potential failure scenarios when using Redis for locking?
- Client crash before `DEL` (fixed with TTL).
- Lock overwrite (use UUID + check).
- Redis node crash (Redlock solves this).
- Network partition (majority quorum in Redlock helps).
- Slow processing = lock expiry before completion.

---

## 13. What are the differences between Redis Pub/Sub and Redis Streams?
| Feature        | Pub/Sub          | Streams                   |
|----------------|------------------|----------------------------|
| Durable?       | No               | Yes                        |
| Replayable?    | No               | Yes                        |
| Delivery Mode  | At-most-once     | At-least-once              |
| Use Cases      | Real-time chats  | Work queues, logging       |

---

## 14. How would you implement a durable work queue in Redis?
- Use Streams.
- Producer: `XADD`.
- Create consumer group: `XGROUP CREATE`.
- Worker: `XREADGROUP`.
- Acknowledge: `XACK`.
- Retry failed tasks with `XPENDING` + `XCLAIM`.

---

## 15. What happens if a consumer in a Redis Stream fails before acknowledging a message?
- Message stays in Pending Entries List (PEL).
- Can be listed using `XPENDING`.
- Reclaimed by another worker using `XCLAIM`.

---

## 16. How does Redis Cluster distribute data across nodes?
- Cluster uses 16,384 hash slots.
- Each node owns a range of slots.
- Key's slot = `CRC16(key) % 16384`.
- Clients connect directly to appropriate node.

---

## 17. What are hash slots in Redis and why are they important?
- Fundamental sharding unit in Redis Cluster.
- Enables predictable and balanced key distribution.
- Used to map keys to nodes.
- Needed for scaling and clustering.

---

## 18. Can a Redis command access data from multiple keys on different nodes?
- No, Redis Cluster disallows cross-slot multi-key commands.
- All keys in a command must belong to the same slot.
- Use hash tags `{}` to force keys into same slot.

---

## 19. Describe how you would implement a fixed-window rate limiter using Redis.
- `INCR` a key per time window (e.g. per minute).
- Set TTL using `EXPIRE` slightly longer than window.
- Block if count exceeds threshold.


---

## 20. What are the limitations of using Redis for rate limiting in high-traffic systems?
- Hot keys may overload a node.
- Fixed window allows bursts.
- Memory overhead from short-lived keys.
- Smoothing/backoff logic must be implemented manually.
- Remote Redis adds latency if rate checking every request.
