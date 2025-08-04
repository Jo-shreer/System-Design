Distributed Locking

Distributed locking controls access to shared resources across multiple nodes.

Why needed:

Prevent conflicts when multiple nodes try to modify the same resource.
Ensure mutual exclusion.
Common Algorithms/Tools:

Chubby (Google): A lock service using consensus (Paxos).
Zookeeper: Provides distributed locks using ephemeral znodes.
Redlock (Redis): Algorithm for distributed locks with Redis instances.

Challenges:
Avoiding deadlocks.
Handling failures of nodes holding locks.
Ensuring locks expire or get released properly.
