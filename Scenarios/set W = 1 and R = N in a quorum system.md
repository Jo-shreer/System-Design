If you set W = 1 and R = N in a quorum system, what does that mean for consistency and availability?

Candidate (strong answer):
If W = 1 and R = N:
Writes are acknowledged as soon as one replica stores the data → fast writes,
but risky if that replica fails before replication.

Reads must check all replicas and reconcile differences → this ensures you always get the most up-to-date value, 
so consistency is very strong.

The downside is that reads are slow and less available, since all replicas must respond. 
If even one replica is down, the read fails.

Summary:

High consistency (because reads compare across all replicas).
Low availability (because reads fail if any replica is down).
Fast writes, slow reads.
