Distributed Caching

Distributed caching stores frequently accessed data in multiple cache servers to reduce latency and improve throughput.

Benefits:
- Faster data access by serving from cache rather than slower databases.
- Reduced load on backend databases.
- Scalability by adding more cache nodes.

Common Architectures:
- **Client-side caching:** Cache maintained on client devices.
- **Server-side caching:** Centralized or distributed cache servers.

Popular Distributed Cache Systems:
- **Redis:** In-memory data store with replication and persistence.
- **Memcached:** Simple, high-performance caching system.
- **Hazelcast:** In-memory data grid with distributed caching.

Challenges:
- Cache consistency with the underlying data store.
- Handling cache invalidation and updates.
- Balancing cache size and eviction policies.

1. Cache consistency (keeping data fresh)
Write-through caching: Every write goes to both DB and cache → keeps cache always consistent but higher latency.
Write-behind caching: Writes go to cache first, then asynchronously to DB → faster but risks inconsistency if cache fails.
Cache-aside (lazy loading): App checks cache first; if a miss, read from DB and update cache. Updates must invalidate or refresh cache entries after a write.
👉 Using Redis, you’d typically combine cache-aside + explicit invalidation when posts/updates/deletes occur.

2. Cache eviction (when cache is full)
LRU (Least Recently Used): Evict least recently accessed item (most common, Redis default).
Other strategies:
LFU (Least Frequently Used): Evict items used least often.
FIFO (First-In, First-Out).

Redis allows configuring eviction policies per use case.

3.Cache invalidation (when underlying data changes)
Explicit invalidation: After DB update, explicitly delete/update the cache entry.
TTL (Time-to-Live): Set expiry times so stale data auto-refreshes.
Pub/Sub invalidation: For distributed caches, use a publish-subscribe channel (e.g., Redis channels) to notify other nodes to drop/update entries.
---
