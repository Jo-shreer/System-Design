you’re designing a global chat application (like WhatsApp). Would you prefer leader-based or leaderless replication, and why?

Candidate (strong response):
For a global chat application, I would prefer leader-based replication.
In chat systems, message ordering is critical. If Alice sends “Hi” and then “How are you?”, Bob should not see them in reverse order.
With a leader, all writes flow through a single node (per chat or per partition), so it’s easier to enforce a total order of messages.
Followers asynchronously replicate from the leader, which gives lower latency for reads while still preserving ordering guarantees.

Why not leaderless?
Leaderless replication (like Dynamo) provides high availability, 
but resolving conflicts (e.g., two users sending messages at the exact same millisecond) would require complex conflict-resolution logic.

That’s okay for something like a shopping cart (where “last write wins” might be fine), but not for real-time ordered conversations.

Real-world example:
WhatsApp uses a leader-based design per chat (each chat is sharded to a primary server), 
which ensures ordering but still scales globally by having many leaders across different shards.
