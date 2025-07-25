# Twitter System Design - End-to-End Component Flows

## Flow 1: User Registration & Authentication Module

### User Registration Flow
```
Client Request → Load Balancer → API Gateway → User Service
    ↓
User Service validates input (email, username uniqueness)
    ↓
User Service → PostgreSQL (insert user record)
    ↓
User Service → Redis (cache user profile)
    ↓
User Service → Kafka (publish user_created event)
    ↓
Background Services consume event:
    - Email Service sends welcome email
    - Analytics Service records new user metric
    - Follow Service creates empty follower/following lists
    ↓
Response sent back to client with user ID and profile data
```

### Authentication Flow
```
Client Login Request → API Gateway → User Service
    ↓
User Service → PostgreSQL (verify credentials)
    ↓
If valid: Generate JWT token with user claims
    ↓
User Service → Redis (store session data with TTL)
    ↓
Return JWT token to client
    ↓
Client stores token and includes in Authorization header for all requests
    ↓
API Gateway validates JWT on each request before routing
```

## Flow 2: Tweet Creation & Distribution Module

### Tweet Posting Flow
```
Client POST /tweets → Load Balancer → API Gateway (JWT validation) → Tweet Service
    ↓
Tweet Service validates tweet (length, content policy)
    ↓
If media attached:
    Tweet Service → Media Service → Object Storage (S3)
    Media Service → Background queue for processing (resize, compress)
    ↓
Tweet Service → PostgreSQL (insert tweet record)
    ↓
Tweet Service → Kafka (publish tweet_created event with tweet_id, user_id)
    ↓
Multiple services consume the event:
    1. Timeline Service → Fan-out process
    2. Search Service → Elasticsearch indexing
    3. Analytics Service → Metrics tracking
    4. Notification Service → Follower notifications
    ↓
Return tweet_id and creation timestamp to client
```

### Fan-out Process (Timeline Distribution)
```
Timeline Service receives tweet_created event
    ↓
Timeline Service → Follow Service (get follower list for tweet author)
    ↓
Decision Engine evaluates follower count:
    - < 1000 followers: PUSH model
    - 1000-100K followers: HYBRID model  
    - > 100K followers: PULL model
    ↓
PUSH Model:
    For each follower:
        Timeline Service → Redis (prepend tweet to follower's timeline cache)
        Timeline Service → Notification Service (real-time notification)
    ↓
PULL Model:
    Timeline Service → Redis (mark user as having new content)
    No immediate timeline updates
    ↓
HYBRID Model:
    Push to active followers (recently online)
    Pull model for inactive followers
```

## Flow 3: Timeline Generation & Retrieval Module

### Home Timeline Request Flow
```
Client GET /timeline → API Gateway → Timeline Service
    ↓
Timeline Service → Redis (check user timeline cache)
    ↓
Cache HIT:
    Return cached timeline with pagination
    Background refresh if cache is > 10 minutes old
    ↓
Cache MISS:
    Timeline Service → Follow Service (get following list)
    ↓
    Timeline Service → Tweet Service (get recent tweets from followees)
    ↓
    Merge and sort tweets by timestamp (last 800 tweets)
    ↓
    Timeline Service → Redis (cache generated timeline, TTL 1 hour)
    ↓
    Return timeline to client
    ↓
Background Timeline Refresh:
    Periodic job updates timelines for active users
    Kafka events trigger selective timeline updates
```

### User Profile Timeline Flow
```
Client GET /users/{id}/tweets → API Gateway → Tweet Service
    ↓
Tweet Service → Redis (check user tweet cache)
    ↓
Cache MISS:
    Tweet Service → PostgreSQL (query tweets by user_id, order by created_at)
    Tweet Service → Redis (cache user tweets, TTL 30 minutes)
    ↓
Apply privacy filters (blocked users, private accounts)
    ↓
Return paginated tweets to client
```

## Flow 4: Real-time Notifications Module

### Like/Retweet/Reply Notification Flow
```
Client POST /tweets/{id}/like → API Gateway → Tweet Service
    ↓
Tweet Service → PostgreSQL (increment like_count, insert like record)
    ↓
Tweet Service → Kafka (publish like_event with user_id, tweet_id, author_id)
    ↓
Notification Service consumes like_event:
    ↓
    Notification Service → User Service (get tweet author details)
    ↓
    Notification Service → Redis (check if user is online via WebSocket)
    ↓
    If online: Send real-time notification via WebSocket
    If offline: 
        → Push Notification Service (mobile push)
        → Email queue (digest email if enabled)
    ↓
    Notification Service → PostgreSQL (store notification record)
```

### WebSocket Connection Management
```
Client connects to WebSocket → Load Balancer (sticky sessions) → Notification Service
    ↓
Notification Service creates connection pool entry
    ↓
Notification Service → Redis (store user_id → connection_id mapping)
    ↓
Heartbeat mechanism maintains connection
    ↓
When notification needed:
    Event Dispatcher → Redis (lookup user connection)
    → WebSocket send notification
    ↓
Connection cleanup on disconnect:
    Remove from Redis mapping
    Clean up connection pool
```

## Flow 5: Search Functionality Module

### Search Request Flow
```
Client GET /search?q=query → API Gateway → Search Service
    ↓
Search Service → Redis (check search result cache with query hash)
    ↓
Cache HIT: Return cached results
Cache MISS:
    ↓
    Search Service → Elasticsearch (execute search query)
    ↓
    Process and rank results:
        - Relevance scoring
        - Recency boost
        - User interaction boost
    ↓
    Search Service → Redis (cache results for 5 minutes)
    ↓
    Search Service → Analytics Service (log search query for trends)
    ↓
    Return search results to client
```

### Search Index Update Flow
```
Tweet Service publishes tweet_created/updated/deleted event → Kafka
    ↓
Search Indexer Service consumes event:
    ↓
    Extract searchable content:
        - Tweet text
        - Hashtags
        - Mentions
        - User information
    ↓
    Search Indexer → Elasticsearch (index/update/delete document)
    ↓
    Search Indexer → Analytics Service (update search trend data)
    ↓
    Index optimization runs periodically for performance
```

## Flow 6: Media Upload & Processing Module

### Media Upload Flow
```
Client POST /media (multipart form) → API Gateway → Media Service
    ↓
Media Service validates file:
    - File type (image/video/gif)
    - File size limits
    - Virus scanning
    ↓
Media Service → Object Storage S3 (upload original file)
    ↓
Generate unique media_id and temporary URL
    ↓
Media Service → Kafka (publish media_uploaded event)
    ↓
Return media_id to client for tweet attachment
    ↓
Background Media Processing:
    Media Worker consumes event
    ↓
    Generate multiple versions:
        - Thumbnails (150x150, 300x300)
        - Compressed versions
        - Different formats (WebP, JPEG)
    ↓
    Upload processed versions to S3
    ↓
    Media Service → CDN (distribute globally)
    ↓
    Update media record with final URLs
```

### Media Serving Flow
```
Client requests media URL → CDN (CloudFlare)
    ↓
CDN checks cache:
    HIT: Serve from edge location
    MISS: 
        CDN → Origin S3 bucket
        Cache at edge for future requests
    ↓
Serve optimized media based on:
    - Device type (mobile/desktop)
    - Connection speed
    - Browser capabilities
```

## Flow 7: Follow/Unfollow Operations Module

### Follow User Flow
```
Client POST /users/{id}/follow → API Gateway → Follow Service
    ↓
Follow Service validates:
    - User exists
    - Not already following
    - Not blocked by target user
    ↓
Follow Service → PostgreSQL (insert follow relationship)
    ↓
Follow Service → Redis (update follower/following caches)
    ↓
Follow Service → Kafka (publish follow_event)
    ↓
Multiple services consume event:
    1. User Service → Update follower/following counts
    2. Timeline Service → Add followed user's tweets to timeline
    3. Notification Service → Notify followed user
    4. Analytics Service → Track follow metrics
    ↓
Return success response to client
```

### Unfollow User Flow
```
Client DELETE /users/{id}/follow → API Gateway → Follow Service
    ↓
Follow Service → PostgreSQL (delete follow relationship)
    ↓
Follow Service → Redis (update caches, remove from lists)
    ↓
Follow Service → Kafka (publish unfollow_event)
    ↓
Timeline Service removes unfollowed user's tweets from timeline
    ↓
User Service decrements follower/following counts
```

## Flow 8: Analytics & Trending Topics Module

### Analytics Data Collection Flow
```
All Services → Kafka (stream events):
    - tweet_created, tweet_liked, tweet_retweeted
    - user_followed, user_unfollowed
    - search_performed, timeline_viewed
    ↓
Analytics Service consumes all events:
    ↓
    Stream Processing (Apache Spark):
        - Real-time aggregations
        - Window-based calculations
        - Anomaly detection
    ↓
    Store in ClickHouse:
        - Time-series metrics
        - User engagement data
        - Geographic trends
    ↓
    Update Redis with real-time metrics for dashboards
```

### Trending Topics Calculation
```
Tweet Stream → Extract hashtags and mentions
    ↓
Time-window counting (last 1 hour, 6 hours, 24 hours)
    ↓
Calculate velocity (mentions per minute)
    ↓
Anomaly Detection:
    - Compare with historical data
    - Identify sudden spikes
    - Filter spam/bot activity
    ↓
Geographic Analysis:
    - Group by user location
    - Identify regional trends
    ↓
Rank trends by:
    - Volume × Velocity × Uniqueness
    ↓
Analytics Service → Redis (cache trending topics by region)
    ↓
API endpoint serves trending topics from cache
```

## Flow 9: Content Moderation Module

### Automated Content Moderation Flow
```
Tweet Service receives new tweet → Content Moderation API
    ↓
AI/ML Analysis Pipeline:
    - Text analysis (hate speech, spam detection)
    - Image analysis (inappropriate content)
    - Link analysis (malicious URLs)
    - User behavior analysis (bot detection)
    ↓
Moderation Score Generated (0-100):
    - 0-30: Auto-approve
    - 31-70: Queue for human review
    - 71-100: Auto-reject or shadow ban
    ↓
Auto-approve: Normal tweet flow continues
Auto-reject: Tweet blocked, user notified
Human Review: Tweet queued for manual moderation
    ↓
Human Moderator Decision:
    Approve → Release tweet to normal flow
    Reject → Permanent block, possible account action
    ↓
Feedback loop updates ML models with moderation decisions
```

## Flow 10: System Monitoring & Health Checks Module

### Monitoring Data Collection Flow
```
All Services → Metrics Collectors:
    - Application metrics (Prometheus)
    - System metrics (CPU, memory, disk)
    - Custom business metrics
    ↓
Time Series Database (InfluxDB) stores all metrics
    ↓
Grafana dashboards visualize:
    - Service health
    - Performance metrics
    - Business KPIs
    ↓
Alert Manager monitors thresholds:
    - Error rate > 1%
    - Response time > 500ms
    - Queue length > 1000
    ↓
PagerDuty notifications for critical alerts
```

### Health Check Flow
```
Load Balancer → Health endpoints (/health) every 30 seconds
    ↓
Each Service checks:
    - Database connectivity
    - Cache availability  
    - Message queue health
    - Dependency service status
    ↓
Return health status:
    - 200 OK: Service healthy
    - 503 Service Unavailable: Service degraded
    ↓
Load Balancer routes traffic only to healthy instances
    ↓
Unhealthy instances automatically removed from rotation
    ↓
Auto-scaling triggers new instances if needed
```

## Flow 11: Disaster Recovery & Backup Module

### Continuous Backup Flow
```
Primary Database → Continuous WAL shipping → Secondary regions
    ↓
Scheduled full backups:
    PostgreSQL → pg_dump → S3 (encrypted, cross-region)
    Redis → RDB snapshots → S3
    ↓
Backup verification:
    Restore to test environment
    Validate data integrity
    ↓
Retention policy:
    Daily backups kept for 30 days
    Weekly backups kept for 1 year
    Monthly backups kept for 3 years
```

### Disaster Recovery Flow
```
Monitoring detects primary region failure
    ↓
Automatic failover process:
    1. DNS updates point to secondary region
    2. Promote read replicas to primary
    3. Update load balancer configuration
    4. Redirect traffic to healthy region
    ↓
Recovery time objective: < 15 minutes
Recovery point objective: < 5 minutes data loss
    ↓
When primary region recovers:
    Reverse replication to sync data
    Planned failback during maintenance window
```

## Flow 12: Cache Management Module

### Cache Population Flow
```
Application requests data → Check Redis cache
    ↓
Cache MISS:
    Application → Database query
    ↓
    Store result in cache with appropriate TTL:
        - User profiles: 1 hour
        - Timeline data: 30 minutes
        - Tweet content: 6 hours
        - Search results: 5 minutes
    ↓
Cache HIT: Return cached data directly
    ↓
Cache warming strategies:
    - Preload popular content
    - Refresh expiring cache in background
    - Predictive caching based on user patterns
```

### Cache Invalidation Flow
```
Data update event (tweet, user profile, follow) → Kafka
    ↓
Cache Invalidation Service consumes event:
    ↓
    Identify affected cache keys:
        - User timeline caches
        - Follower list caches
        - Search result caches
    ↓
    Redis operations:
        - DELETE expired keys
        - SET updated values
        - EXPIRE keys with new TTL
    ↓
    Multi-region cache synchronization for global consistency
```

## Performance Metrics Summary

### Read Operations (90% of traffic)
- Timeline requests: 200ms average (Redis cache)
- Search queries: 150ms average (Elasticsearch + cache)
- Profile views: 100ms average (Redis cache hit)
- Media serving: 50ms average (CDN)

### Write Operations (10% of traffic)
- Tweet creation: 500ms average (includes fan-out)
- Follow operations: 200ms average
- Like/retweet: 100ms average
- Media upload: 2-5 seconds (depending on size)

### System Throughput
- Peak tweets/second: 10,000
- Peak timeline requests/second: 1,000,000
- Peak search queries/second: 50,000
- Active WebSocket connections: 10,000,000

This comprehensive flow analysis demonstrates how each component operates independently while maintaining system-wide consistency and performance through event-driven architecture and strategic caching.
