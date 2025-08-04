#CAP?
The CAP theorem states that in a distributed system, you can only guarantee two out of the following three properties simultaneously:
Consistency (C): Every read receives the most recent write or an error.
Availability (A): Every request receives a (non-error) response, without guarantee that it contains the most recent write.
Partition tolerance (P): The system continues to operate despite arbitrary message loss or failure of part of the system (network partitions).
The reason you can’t have all three at once is because during a network partition (P), the system must choose between:
Consistency: Ensure all nodes see the same data, which might require some nodes to become unavailable until they can sync again, sacrificing availability.
Availability: Ensure every request gets a response, even if it means some nodes return stale data, sacrificing consistency.
Since network partitions are inevitable in distributed systems, partition tolerance is a must, and the system trades off between 
consistency and availability depending on use case.


#what a distributed lock is, why it’s needed, and how you might implement one using tools like Zookeeper or Redis?
What is a distributed lock?
A distributed lock is a mechanism that ensures only one process (out of many running on different machines) can access a shared resource at a time.

Example:
Imagine multiple services trying to update the same database row or write to the same file. Without coordination, they could overwrite each other’s changes.
A distributed lock prevents that by allowing only one service to "hold the lock" at any time.

Why is it needed?
To prevent race conditions in distributed systems
To ensure mutual exclusion across nodes
To maintain data consistency when multiple processes interact with the same resource

How to implement it?
Using Zookeeper
Zookeeper is often used as a lock service.
Each client creates an ephemeral znode (temporary node).
The client with the "lowest sequential node" gets the lock.
If a client holding the lock crashes, its znode is deleted automatically → lock is released.

Using Redis (popular with cloud systems)
Redis can be used to set a lock with SET resource value NX PX timeout
NX = only set if not exists
PX timeout = lock automatically expires after some milliseconds
This prevents deadlocks.
The Redlock algorithm (from Redis Labs) is often used for distributed locking across multiple Redis instances.

Leader Election in Distributed Systems
What it is:
Leader election is the process of choosing one node (server) in a distributed system to act as the coordinator or decision-maker.

Why it’s important:
To avoid conflicts when multiple nodes try to act simultaneously
To coordinate writes, locks, or metadata updates
To handle failures — if a leader crashes, a new one must be elected.

Examples of Leader Election Algorithms
Raft Consensus Algorithm
Raft divides the cluster into: Leader, Followers, and Candidates.
The leader handles all writes. Followers just replicate logs.
If a leader fails, followers start an election:
A follower becomes a candidate.
It requests votes from other nodes.
If it gets a majority → it becomes the new leader.
✅ Raft is designed to be understandable and practical compared to Paxos.

Zookeeper Leader Election
Each node creates an ephemeral sequential znode in Zookeeper.
The node with the lowest sequence number becomes the leader.
If the leader node fails, its znode disappears, and the next lowest becomes leader automatically.
✅ Used in systems like Kafka for broker coordination.




