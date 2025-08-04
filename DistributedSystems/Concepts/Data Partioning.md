Data Partitioning (Sharding)

Partitioning means splitting data across multiple nodes to improve scalability and performance.

Types of Partitioning:

1. **Horizontal Partitioning (Sharding)**  
   - Each shard holds a subset of rows.  
   - Example: User data partitioned by geographic region.

2. **Vertical Partitioning**  
   - Each partition holds different columns of the same table.  
   - Example: Separating user profile info from login credentials.

3. **Directory-Based Partitioning**  
   - Uses a lookup table to find which partition stores the data.

---

Shard Key Selection

Choosing the right shard key is critical for even data distribution and query efficiency.

Good shard keys:  
- Have high cardinality (many unique values).  
- Are frequently used in queries.  
- Avoid hotspots (too much data or traffic on one shard).

---

Replication Strategies in Sharding

- **Master-Slave:** One master handles writes; slaves handle reads.  
- **Master-Master:** Multiple masters accept writes; more complex consistency.  
- **Leader-Follower:** Similar to master-slave but with leader election.

---

Partition Tolerance and Network Partitions

A network partition splits nodes into isolated groups. Systems must handle this gracefully.

- **Partition Tolerance:** The system continues functioning despite partitions.  
- This forces trade-offs in CAP theorem between Consistency and Availability.

---

Consistency in Partitioned Systems

- Strong consistency is harder to maintain.  
- Eventual consistency often chosen for availability and partition tolerance.  
- Techniques like vector clocks and versioning help track data changes.

---


