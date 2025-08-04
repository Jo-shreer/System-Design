Synchronous Replication:

The leader waits until data is written to all required replicas before acknowledging the client.
Ensures strong consistency because all replicas have the same data at commit time.
Tradeoff: Higher latency and potential unavailability if replicas are slow.
Example use case: Banking/financial systems — a money transfer must be fully replicated before confirming success.

Asynchronous Replication:
The leader acknowledges the client as soon as it writes locally, and sends updates to replicas in the background.
Ensures low latency and high availability, but replicas may be temporarily out-of-date.
Tradeoff: Risk of data loss if the leader fails before replicas catch up.

Example use case: Social media feeds, analytics dashboards — where slight lag is acceptable.

If you’re designing a global e-commerce platform (like Amazon), which replication strategy would you choose for:
The shopping cart service
The order payment/transaction service

Model Answer:
Shopping Cart Service → Asynchronous Replication
The shopping cart is usually not mission-critical in terms of strong consistency.
If a user adds an item and it shows up with a slight delay on another device, that’s acceptable.
Using asynchronous replication here improves latency and scalability since carts are often read-heavy and updated frequently.
Many systems (like Amazon) store the shopping cart in a leaderless eventually consistent store (like DynamoDB).

Order Payment / Transaction Service → Synchronous Replication
Payments require strong consistency — once you charge a customer, the system must guarantee the order exists and the transaction is recorded everywhere.
Synchronous replication ensures no double-charging or missing transactions if a node fails.
Tradeoff: higher latency, but correctness outweighs performance.

Summary:

Shopping cart → asynchronous replication (availability > strict consistency).
Order/payment → synchronous replication (consistency > latency).

