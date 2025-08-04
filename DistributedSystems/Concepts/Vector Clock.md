Vector Clocks

Vector clocks are a mechanism to capture causal relationships between events in a distributed system.

How it works:
- Each node maintains a vector (array) of counters, one per node.
- When a node performs an event, it increments its own counter.
- When nodes exchange messages, they update their vectors by taking the element-wise maximum.
- This helps determine if one event happened before another, or if events are concurrent (no causal order).

Use Cases:
- Conflict resolution in eventually consistent databases.
- Tracking versions of data objects.
- Detecting causality and concurrency.

---

Distributed Locking

Distributed locking controls access to shared resources across multiple nodes.

Why needed:
- Prevent conflicts when multiple nodes try to modify the same resource.
- Ensure mutual exclusion.

Common Algorithms/Tools:
- **Chubby (Google):** A lock service using consensus (Paxos).
- **Zookeeper:** Provides distributed locks using ephemeral znodes.
- **Redlock (Redis):** Algorithm for distributed locks with Redis instances.

Challenges:
- Avoiding deadlocks.
- Handling failures of nodes holding locks.
- Ensuring locks expire or get released properly.

---

Real-World Distributed Systems Examples

1. **Kubernetes**  
   - Container orchestration platform.  
   - Uses etcd (Raft consensus) for cluster state management.  
   - Handles scheduling, scaling, and health monitoring.

2. **Apache Cassandra**  
   - Distributed NoSQL database.  
   - Uses consistent hashing for partitioning.  
   - Supports eventual consistency with tunable consistency levels.

3. **Google Spanner**  
   - Globally distributed SQL database.  
   - Strong consistency using TrueTime API (GPS + atomic clocks).  
   - Uses Paxos for consensus.

---

Would you like me to explain any of these systems in detail or cover topics like distributed caching or monitoring next?
