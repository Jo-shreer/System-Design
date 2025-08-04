Quorum Reads/Writes in Distributed Databases
In a replicated distributed system (like Cassandra or DynamoDB), 
data is stored on multiple nodes (replicas). 
To maintain consistency while still allowing high availability, these systems use quorums.

Quorum Write:
A write is considered successful once it has been written to a majority of replicas (not all).
Example: If data is replicated to 3 nodes, a quorum write might mean acknowledgment from at least 2 nodes.

Quorum Read:
A read request checks a majority of replicas to ensure it gets the most up-to-date value.
Example: If 3 replicas exist, the system reads from 2 of them and reconciles differences.

Formula:
𝑅
+
𝑊
>
𝑁
R+W>N

R = number of replicas read
W = number of replicas written
N = total number of replicas
If this holds, then reads and writes will always overlap on at least one replica, guaranteeing you see the latest write.

Why use it?

It balances consistency, availability, and performance.
Instead of waiting for all replicas (slow), the system only waits for a quorum, 
which is faster but still ensures strong-enough consistency.

Examples:

Cassandra, DynamoDB: Use quorum reads/writes with tunable consistency (you can choose ONE, QUORUM, or ALL).

