1. Distributed Transactions (Advanced Concepts)

Recap: Distributed transactions ensure that a group of operations across multiple nodes either all succeed or all fail to keep data consistent.

Advanced Concepts:

- **Nested Transactions:** Transactions within transactions, allowing partial commits at sub-transaction levels. Useful in complex workflows.

- **Compensating Transactions:** When rollback is not possible, compensating transactions undo the effects of previous steps.
- Common in long-running distributed business processes.

- **Two-Phase Locking (2PL):** Ensures serializability by acquiring locks during the transaction and releasing them only after commit or abort.

- **Optimistic Concurrency Control:** Transactions proceed without locking; conflicts detected at commit time, and conflicting transactions are rolled back.

- **Distributed Snapshot:** Captures a global consistent state without stopping the system. Algorithms like Chandy-Lamport help.

Example Scenario:

Imagine an e-commerce system where placing an order involves:

- Deducting inventory from warehouse A.
- Charging customer’s payment.
- Updating order status.

These steps span different services/nodes and must all succeed or fail together.

If charging payment succeeds but inventory deduction fails, compensating transaction might refund the charge to maintain consistency.

---


