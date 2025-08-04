

Fault Tolerance in Distributed Systems

Fault tolerance means the system keeps working correctly even if some nodes or components fail.

Types of Failures:
- Crash Failure: Node stops working.
- Omission Failure: Messages lost or not sent.
- Timing Failure: Messages delayed.
- Byzantine Failure: Nodes behave arbitrarily or maliciously.

Techniques for Fault Tolerance:
- Replication: Keep copies of data on multiple nodes.
- Heartbeats: Nodes regularly send signals to show they’re alive.
- Failover: Automatically switch to a backup node if one fails.
- Checkpointing: Save state periodically to recover after failure.
- Consensus Algorithms: Ensure agreement despite failures.

---

Replication

Replication means storing copies of data or services on multiple nodes to improve reliability and performance.

Types:
- Synchronous Replication: Updates happen on all copies before acknowledging success.
- Asynchronous Replication: Updates propagate later, increasing speed but risking inconsistency.

Benefits:
- High availability (system still works if some nodes fail).
- Load balancing (requests spread across replicas).
- Faster data access (serve requests from nearest replica).

Challenges:
- Keeping replicas consistent.
- Handling network partitions (nodes can’t talk to each other).

---

CAP Theorem

The CAP theorem states that in a distributed system, you can only guarantee two of the following three at the same time:

- Consistency (C): All nodes see the same data at the same time.
- Availability (A): Every request receives a response.
- Partition Tolerance (P): The system continues to work despite network splits.

Implications:
- You must choose what to prioritize based on your system’s needs.
- Most distributed systems choose Partition Tolerance and either Consistency or Availability.

---

Consistency Models

Different ways to define how data is seen by nodes.

- Strong Consistency: After an update, all nodes see the same data immediately.
- Eventual Consistency: Updates propagate eventually; nodes might see stale data temporarily.
- Causal Consistency: If one operation depends on another, all nodes see them in that order.
- Weak Consistency: No guarantees on order or timing of updates.

---

Would you like me to explain examples of distributed databases, or how distributed transactions work next?
