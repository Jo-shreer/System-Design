📤 TWITTER SYSTEM DESIGN — DATA FLOW EXPLANATION

1️⃣ USER ACTION
- User opens app or website, sends a request (e.g., post tweet, follow user, load timeline)

2️⃣ API GATEWAY
- Receives the request
- Authenticates user (OAuth2/JWT)
- Enforces rate limits, logs request
- Routes to the correct microservice via internal service mesh

3️⃣ LOAD BALANCER
- Distributes incoming traffic across multiple instances of each microservice
- Uses health checks to avoid sending traffic to unhealthy services
- Supports auto-scaling during peak traffic

4️⃣ MICROSERVICE LAYER

🟡 A. Post Tweet
- Request hits **Tweet Service**
- Tweet content is stored in **Cassandra** (sharded by tweet_id)
- Tweet ID is published to **Kafka (new_tweet topic)**

🟢 B. Kafka Fan-Out (Async)
- **Timeline Worker** listens to `new_tweet`
- Retrieves followers of the tweeting user (from Redis/PostgreSQL)
- Pushes the tweet ID to each follower's timeline in **Redis** or **Cassandra**

🔵 C. Timeline Fetch
- User opens their feed → request goes to **Timeline Service**
- Tries to read timeline data from **Redis cache**
- On cache miss, fetches from **Cassandra backup**, rebuilds cache

🔴 D. Follow/Unfollow
- Request goes to **User Service**
- Updates PostgreSQL follow graph
- If "follow", triggers **Kafka (new_follow topic)** to update follower timelines

🟠 E. Media Upload (Optional)
- Images/videos uploaded are sent to **S3**
- Accessed via signed URLs and served through **CDN (e.g., CloudFront)**

🟣 F. Search Tweet
- Search query is routed to **Search Service**
- Powered by **Elasticsearch**, indexes tweet content & hashtags
- Returns ranked, full-text results

5️⃣ ASYNC PROCESSING
- Kafka ensures reliable delivery of events like tweets, follows, likes
- Consumers process events independently without blocking user interactions
- Retry-safe due to idempotency

6️⃣ CACHING
- **Redis** stores:
  - Hot tweets
  - User timelines
  - Profile info
  - Follower lists
- Reduces DB load and speeds up reads

7️⃣ MONITORING & FAULT TOLERANCE
- Every service exports metrics to **Prometheus**
- Traces requests via **OpenTelemetry / Jaeger**
- Logs to **ELK Stack**
- Alerts sent to **PagerDuty/OpsGenie**
- Failures handled via:
  - Auto-healing (Kubernetes)
  - Replication (Redis, Cassandra, Kafka)
  - Circuit breakers, retries, and timeouts

8️⃣ SCALE & FAILOVER
- Stateless services auto-scale with traffic
- Data replicated across AZs and regions
- Load balancers + DNS handle region failover

✅ RESULT: Fast, scalable, fault-tolerant architecture supporting millions of users and tweets/day.
