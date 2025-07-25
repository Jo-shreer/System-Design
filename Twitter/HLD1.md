# Twitter System Design - High Level Design

## 1. Requirements Analysis

### Functional Requirements
- **User Management**: Registration, authentication, profile management
- **Tweet Operations**: Create, read, delete tweets (max 280 characters)
- **Timeline Generation**: Home timeline, user timeline
- **Follow System**: Follow/unfollow users
- **Engagement**: Like, retweet, reply to tweets
- **Search**: Search tweets and users
- **Notifications**: Real-time notifications for interactions
- **Media Support**: Images, videos, GIFs in tweets

### Non-Functional Requirements
- **Scale**: 500M daily active users, 400M tweets/day
- **Read Heavy**: 100:1 read to write ratio
- **Availability**: 99.9% uptime
- **Latency**: Timeline load < 200ms, tweet posting < 1s
- **Consistency**: Eventual consistency acceptable
- **Global Distribution**: Worldwide user base

## 2. Capacity Estimation

### Storage Requirements
- **Users**: 1B users × 1KB = 1TB
- **Tweets**: 400M tweets/day × 365 days × 5 years × 300 bytes = ~220TB
- **Media**: 20% tweets have media, avg 200KB = ~58TB/year
- **Total**: ~300TB for 5 years

### Bandwidth Requirements
- **Write**: 400M tweets/day = 4,630 tweets/second
- **Read**: 100:1 ratio = 463K timeline requests/second
- **Peak Load**: 3x average = 1.4M requests/second

### Cache Requirements
- **Timeline Cache**: 500M users × 800 tweets × 280 bytes = ~112TB
- **Hot Data**: 20% of data accessed 80% of time = ~22TB active cache

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  CLIENT LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [Web App]    [iOS App]    [Android App]    [Mobile Web]    [Desktop App]      │
│      │            │             │              │                │              │
│      └────────────┼─────────────┼──────────────┼────────────────┘              │
└─────────────────────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                   CDN LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│        [CloudFlare CDN] ◄──── Static Assets, Media Files, Edge Cache           │
└─────────────────────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               LOAD BALANCER                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│    [AWS ALB] ──── Geographic Distribution ──── DDoS Protection ──── SSL Term   │
└─────────────────────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               API GATEWAY                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [Kong/AWS API Gateway] ── Rate Limiting ── Authentication ── Request Routing  │
└─────────────────────────────────────────────────────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             MICROSERVICES LAYER                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │User Service │  │Tweet Service│  │Timeline Svc │  │Follow Svc   │            │
│  │- Register   │  │- Create     │  │- Generate   │  │- Follow     │            │
│  │- Auth       │  │- Read       │  │- Home TL    │  │- Unfollow   │            │
│  │- Profile    │  │- Delete     │  │- User TL    │  │- Get Lists  │            │
│  │- Sessions   │  │- Media      │  │- Push/Pull  │  │- Suggestions│            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Search Svc   │  │Notification │  │Media Svc    │  │Analytics Svc│            │
│  │- Tweet      │  │- Push       │  │- Upload     │  │- Metrics    │            │
│  │- User       │  │- WebSocket  │  │- Process    │  │- Trends     │            │
│  │- Trending   │  │- Email      │  │- CDN Sync   │  │- Reports    │            │
│  │- Elasticsearch│ │- Mobile     │  │- Thumbnails │  │- ML Models  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
          │                │                │                │
          ▼                ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CACHING LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Redis Cluster│  │Memcached    │  │Application  │  │Query Cache  │            │
│  │- Timeline   │  │- Sessions   │  │Cache        │  │- DB Results │            │
│  │- User Data  │  │- User Prefs │  │- Hot Tweets │  │- Aggregates │            │
│  │- Hot Data   │  │- Temp Data  │  │- Profiles   │  │- Counts     │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
          │                                                 │
          ▼                                                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            MESSAGE QUEUE LAYER                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Kafka Cluster│  │RabbitMQ     │  │Event Stream │  │Dead Letter  │            │
│  │- Tweet       │  │- Notifs     │  │- Real-time  │  │Queue        │            │
│  │- Timeline    │  │- Email      │  │- Analytics  │  │- Failed     │            │
│  │- Follow      │  │- Push       │  │- Search Idx │  │- Retry      │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
          │                                                 │
          ▼                                                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          BACKGROUND SERVICES                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Timeline     │  │Media        │  │Search Index │  │ML/Analytics │            │
│  │Generator    │  │Processor    │  │Builder      │  │Pipeline     │            │
│  │- Fan-out    │  │- Resize     │  │- Real-time  │  │- Recommend  │            │
│  │- Batch Jobs │  │- Compress   │  │- Batch Idx  │  │- Trends     │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Cleanup Jobs │  │Notification │  │Backup &     │  │Monitoring   │            │
│  │- Old Data   │  │Dispatcher   │  │Archive      │  │& Alerting   │            │
│  │- Cache Evict│  │- Queue Proc │  │- S3 Archive │  │- Health     │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
          │                                                 │
          ▼                                                 ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             DATABASE LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │PostgreSQL   │  │Cassandra    │  │Elasticsearch│  │ClickHouse   │            │
│  │Master/Slave │  │Cluster      │  │Cluster      │  │Cluster      │            │
│  │- Users      │  │- Tweets     │  │- Search Idx │  │- Analytics  │            │
│  │- Follows    │  │- Timelines  │  │- User Idx   │  │- Events     │            │
│  │- Metadata   │  │- Time Series│  │- Trends     │  │- Metrics    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐                   ┌─────────────┐  ┌─────────────┐            │
│  │Object       │                   │Graph DB     │  │Time Series  │            │
│  │Storage      │                   │(Neo4j)      │  │DB (InfluxDB)│            │
│  │- Media Files│                   │- Social     │  │- Metrics    │            │
│  │- Avatars    │                   │- Graph      │  │- Performance│            │
│  │- Backups    │                   │- Recommend  │  │- Monitoring │            │
│  └─────────────┘                   └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL SERVICES & INTEGRATIONS                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [Push Notification Services] [Email Services] [SMS Gateways] [Payment APIs]  │
│  [Content Moderation APIs]    [Image/Video Processing]       [ML/AI Services] │
│  [Geolocation Services]       [URL Shorteners]               [Third-party Auth]│
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                         MONITORING & OBSERVABILITY                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [Datadog/New Relic] [ELK Stack] [Prometheus/Grafana] [Jaeger/Zipkin]         │
│  [PagerDuty Alerts]  [APM Tools] [Error Tracking]     [Performance Monitoring]│
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 4. Core Services

### User Service
- **Responsibility**: User registration, authentication, profile management
- **Database**: User profiles, relationships, authentication tokens
- **Cache**: User profile cache, session cache

### Tweet Service
- **Responsibility**: Tweet CRUD operations, media handling
- **Database**: Tweet content, metadata, media references
- **Storage**: Object storage for media files

### Timeline Service
- **Responsibility**: Generate and serve user timelines
- **Strategies**: 
  - Push model for users with few followers
  - Pull model for celebrities with millions of followers
  - Hybrid approach for most users

### Follow Service
- **Responsibility**: Manage follower/following relationships
- **Database**: Adjacency list or matrix for relationships
- **Cache**: Follower lists, following lists

### Notification Service
- **Responsibility**: Real-time notifications for user interactions
- **Technology**: WebSocket connections, push notifications
- **Queue**: Message broker for reliable delivery

### Search Service
- **Responsibility**: Search tweets and users
- **Technology**: Elasticsearch for full-text search
- **Indexing**: Real-time tweet indexing, user search

## 5. Database Design

### User Table
```sql
Users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(255) UNIQUE,
    password_hash VARCHAR(255),
    display_name VARCHAR(100),
    bio TEXT,
    profile_image_url VARCHAR(500),
    created_at TIMESTAMP,
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0
)
```

### Tweet Table
```sql
Tweets (
    tweet_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT(280),
    media_urls JSON,
    created_at TIMESTAMP,
    like_count INT DEFAULT 0,
    retweet_count INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    parent_tweet_id BIGINT NULL, -- for replies
    INDEX(user_id, created_at),
    INDEX(created_at)
)
```

### Follows Table
```sql
Follows (
    follower_id BIGINT,
    followee_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY(follower_id, followee_id),
    INDEX(follower_id),
    INDEX(followee_id)
)
```

### Timeline Cache Schema (Redis)
```
Key: timeline:{user_id}
Value: [tweet_id1, tweet_id2, ...] (sorted by timestamp)
TTL: 24 hours
```

## 6. Timeline Generation Strategies

### Push Model (Fan-out on Write)
- **When**: User posts a tweet
- **Process**: Write tweet to all followers' timelines immediately
- **Pros**: Fast reads, pre-computed timelines
- **Cons**: Expensive writes for users with many followers
- **Best For**: Regular users with < 10K followers

### Pull Model (Fan-out on Read)
- **When**: User requests timeline
- **Process**: Query tweets from all followees and merge
- **Pros**: Efficient writes, handles celebrities well
- **Cons**: Slow reads, expensive timeline generation
- **Best For**: Celebrities with millions of followers

### Hybrid Approach
- **Strategy**: Push for most users, pull for celebrities
- **Implementation**: 
  - Maintain threshold (e.g., 1M followers)
  - Pre-compute timelines for regular users
  - Generate celebrity timelines on-demand
  - Cache celebrity timelines aggressively

## 7. Caching Strategy

### Cache Layers
1. **CDN**: Static content, media files
2. **Application Cache**: User profiles, hot tweets
3. **Timeline Cache**: Pre-computed user timelines
4. **Database Cache**: Query result caching

### Cache Policies
- **User Profiles**: LRU, 1-hour TTL
- **Timelines**: Write-through, 24-hour TTL
- **Hot Tweets**: Write-behind, 6-hour TTL
- **Media**: CDN caching, 30-day TTL

## 8. Technology Stack

### Frontend
- **Web**: React/Vue.js with PWA capabilities
- **Mobile**: Native iOS/Android apps
- **Real-time**: WebSocket for live updates

### Backend Services
- **API Gateway**: Kong/AWS API Gateway
- **Microservices**: Node.js/Java Spring Boot
- **Message Queue**: Apache Kafka/RabbitMQ
- **Search**: Elasticsearch
- **Cache**: Redis/Memcached

### Databases
- **Primary**: PostgreSQL (ACID compliance)
- **Timeline Storage**: Cassandra (distributed, scalable)
- **Analytics**: ClickHouse (time-series data)
- **Search Index**: Elasticsearch

### Infrastructure
- **Cloud**: AWS/GCP multi-region deployment
- **CDN**: CloudFlare/AWS CloudFront
- **Monitoring**: Datadog/New Relic
- **Load Balancer**: AWS ALB/NGINX

## 9. Scalability Considerations

### Horizontal Scaling
- **Database Sharding**: Shard by user_id or tweet_id
- **Service Partitioning**: Separate services by domain
- **Load Distribution**: Geographic load balancing

### Data Partitioning
- **User Data**: Shard by user_id hash
- **Tweet Data**: Shard by tweet_id or user_id
- **Timeline Data**: Distribute across multiple Redis clusters

### Performance Optimizations
- **Read Replicas**: Multiple read replicas for timeline queries
- **Connection Pooling**: Database connection optimization
- **Async Processing**: Background jobs for heavy operations
- **Batch Operations**: Bulk inserts and updates

## 10. Security & Reliability

### Security Measures
- **Authentication**: OAuth 2.0, JWT tokens
- **Authorization**: Role-based access control
- **Rate Limiting**: API rate limits, DDoS protection
- **Data Encryption**: Encryption at rest and in transit

### Reliability Features
- **Circuit Breakers**: Prevent cascade failures
- **Graceful Degradation**: Partial service availability
- **Backup & Recovery**: Regular backups, point-in-time recovery
- **Health Checks**: Service health monitoring

### Monitoring & Alerting
- **Metrics**: Response time, throughput, error rates
- **Logging**: Centralized logging with ELK stack
- **Alerts**: Threshold-based alerting for critical metrics
- **Dashboards**: Real-time system monitoring

## 11. Trade-offs & Considerations

### Consistency vs Availability
- **Choice**: Eventual consistency for better availability
- **Impact**: Slight delays in timeline updates acceptable
- **Mitigation**: Strong consistency for critical operations (payments)

### Storage vs Computation
- **Timeline Generation**: Pre-computed vs on-demand
- **Decision**: Hybrid approach based on user type
- **Trade-off**: Storage cost vs computation latency

### Global Distribution
- **Challenge**: Cross-region data synchronization
- **Solution**: Regional data centers with eventual consistency
- **Optimization**: Edge caching for global performance

This design provides a scalable, reliable foundation for a Twitter-like social media platform that can handle hundreds of millions of users while maintaining good performance and user experience.
