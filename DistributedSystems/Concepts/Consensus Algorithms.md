What is the problem?
In a distributed system, different nodes might have different views of the system state because:
Messages can be delayed or lost.
Some nodes might crash.
Nodes might join or leave the system dynamically.
Despite this, nodes need to agree on the same value (e.g., who is the leader, or what is the committed transaction) to keep the system consistent.

Why is consensus hard?
Because there is no shared memory and communication is unreliable. Plus, some nodes may fail or act maliciously.
The consensus algorithm makes sure that:
Safety: No two nodes decide on different values.
Liveness: Eventually, nodes will decide on some value.

Classic Example: Leader Election
In many distributed systems, you want to pick a leader node that coordinates actions. 
Consensus algorithms help nodes agree on who that leader is.

Popular Consensus Algorithms:
1. Paxos
Developed by Leslie Lamport.
Paxos is often called “the hardest thing to understand in distributed systems.”
Works by electing a value in a fault-tolerant way.
It uses a set of roles: Proposers, Acceptors, and Learners.
Proposers suggest values.
Acceptors agree to those values.
Learners find out the decided value.
Guarantees safety even if some nodes fail.
Can be complex to implement.

How Paxos works (high level):
A proposer picks a number and proposes a value.
Acceptors respond with promises.
If a majority of acceptors promise, the proposer sends an accept request.
Acceptors accept the value.
Learners learn the value once accepted by a majority.

2. Raft
Raft consensus algorithm is designed to achieve consensus across distributed nodes by electing a single leader that manages the replicated log.

Roles:
Leader: Handles all client requests (writes), appends entries to its log, and replicates them to followers.
Followers: Passive nodes that replicate the leader’s log entries and respond to leader’s heartbeats.
Candidates: When a follower times out without hearing from a leader, it becomes a candidate and starts an election.

Leader Election:
A candidate requests votes from other nodes.
If it gets a majority, it becomes the leader.
This ensures only one leader at a time, preventing split-brain scenarios.

Log Replication:
The leader appends new entries to its log and sends AppendEntries RPCs to followers.
Followers write the entries to their logs and acknowledge.
Once a majority acknowledges, the leader commits the entries.

Handling Leader Failures:
If the leader crashes or becomes unreachable, followers timeout.
One follower becomes a candidate and starts a new election.
A new leader is elected to continue operation.

Why use consensus algorithms?
Without consensus, distributed systems risk split-brain scenarios (different nodes believing different truths), 
leading to data corruption or inconsistent behavior.
