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
