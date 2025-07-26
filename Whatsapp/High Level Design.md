# WhatsApp System Design - Complete Architecture & Flows

## 1. Requirements Analysis

### Functional Requirements
- **Messaging**: Send/receive text messages in real-time
- **Media Sharing**: Photos, videos, documents, voice messages
- **Group Chat**: Create groups, add/remove members, group admin features
- **Voice/Video Calls**: One-on-one and group calls
- **User Management**: Registration with phone number, profile management
- **Message Status**: Sent, delivered, read receipts
- **Online Presence**: Last seen, online status
- **Message Encryption**: End-to-end encryption for security
- **Message History**: Store and sync chat history across devices
- **Push Notifications**: Notify users of new messages when offline

### Non-Functional Requirements
- **Scale**: 2B+ active users, 100B+ messages/day
- **Availability**: 99.9% uptime globally
- **Latency**: Message delivery < 100ms
- **Consistency**: Strong consistency for message ordering
- **Security**: End-to-end encryption, secure authentication
- **Global Distribution**: Multi-region deployment
- **Real-time**: WebSocket connections for instant messaging

## 2. Capacity Estimation

### Storage Requirements
- **Users**: 2B users × 1KB = 2TB
- **Messages**: 100B messages/day × 365 days × 2 years × 1KB = ~73PB
- **Media**: 30% messages have media, avg 500KB = ~55PB/year
- **Voice Messages**: 10% messages, avg 50KB = ~1.8PB/year
- **Total**: ~130PB for 2 years

### Bandwidth Requirements
- **Messages**: 100B messages/day = 1.16M messages/second
- **Peak Load**: 3x average = 3.5M messages/second
- **Media Transfer**: 30% × 500KB = ~580GB/second
- **Voice Calls**: 100M concurrent calls × 64kbps = ~6.4TB/second

### Active Connections
- **Concurrent Users**: 1B users online simultaneously
- **WebSocket Connections**: 1B persistent connections
- **Connection Servers**: 1M connections per server = 1000 servers minimum

## 3. Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  CLIENT LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│    [iOS App]    [Android App]    [Web App]    [Desktop App]    [WhatsApp Web]  │
│        │             │              │              │               │           │
│        └─────────────┼──────────────┼──────────────┼───────────────┘           │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               EDGE LAYER                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│        [Global CDN] ──── Media Delivery ──── [Edge Servers] ──── Geo Routing   │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             LOAD BALANCER                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│     [L4 Load Balancer] ──── Connection Balancing ──── [L7 Load Balancer]       │
│           │                                                    │                │
│    WebSocket Traffic                                    HTTP/API Traffic        │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            API GATEWAY                                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [API Gateway] ── Auth ── Rate Limiting ── Protocol Translation ── Routing     │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        CONNECTION MANAGEMENT LAYER                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │WebSocket    │  │Connection   │  │Presence     │  │Session      │            │
│  │Manager      │  │Pool         │  │Service      │  │Manager      │            │
│  │- Maintain   │  │- 1M conn    │  │- Online     │  │- User       │            │
│  │- Route msgs │  │  per server │  │- Last seen  │  │  sessions   │            │
│  │- Heartbeat  │  │- Load bal   │  │- Typing     │  │- Multi dev  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          CORE SERVICES LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │User Service │  │Message Svc  │  │Group Svc    │  │Media Svc    │            │
│  │- Registration│  │- Send/Recv  │  │- Create     │  │- Upload     │            │
│  │- Auth       │  │- History    │  │- Members    │  │- Process    │            │
│  │- Profile    │  │- Encryption │  │- Admin      │  │- Compress   │            │
│  │- Contacts   │  │- Status     │  │- Broadcast  │  │- Thumbnails │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Call Service │  │Notification │  │Analytics    │  │Security     │            │
│  │- Voice      │  │- Push       │  │- Metrics    │  │- Encryption │            │
│  │- Video      │  │- FCM/APNs   │  │- Usage      │  │- Spam Det   │            │
│  │- Conference │  │- WebSocket  │  │- Reports    │  │- Content    │            │
│  │- Recording  │  │- Email      │  │- ML Models  │  │- Moderation │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           CACHING LAYER                                        │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Redis Cluster│  │Memcached    │  │Application  │  │CDN Cache    │            │
│  │- Messages   │  │- Sessions   │  │Cache        │  │- Media      │            │
│  │- User Data  │  │- Temp Data  │  │- Contacts   │  │- Static     │            │
│  │- Groups     │  │- OTP Codes  │  │- Groups     │  │- Thumbnails │            │
│  │- Presence   │  │- Rate Limit │  │- Metadata   │  │- Documents  │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MESSAGE QUEUE LAYER                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Kafka        │  │RabbitMQ     │  │Event Stream │  │Dead Letter  │            │
│  │- Messages   │  │- Push Notif │  │- Real-time  │  │Queue        │            │
│  │- Events     │  │- Media Proc │  │- Analytics  │  │- Failed     │            │
│  │- Analytics  │  │- Callbacks  │  │- Audit Log  │  │- Retry      │            │
│  │- Logs       │  │- Async Jobs │  │- Monitoring │  │- Manual Rev │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         BACKGROUND SERVICES                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Media        │  │Message      │  │Backup &     │  │Analytics    │            │
│  │Processor    │  │Delivery     │  │Archive      │  │Pipeline     │            │
│  │- Compress   │  │- Retry      │  │- Chat Export│  │- Usage Stats│            │
│  │- Transcode  │  │- Route      │  │- S3 Archive │  │- ML Training│            │
│  │- Thumbnail  │  │- Batch      │  │- Compliance │  │- Insights   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Cleanup Jobs │  │Health Check │  │Security     │  │Monitoring   │            │
│  │- Old Media  │  │- Services   │  │Scanner      │  │& Alerting   │            │
│  │- Logs       │  │- DB Health  │  │- Malware    │  │- Metrics    │            │
│  │- Cache Evict│  │- Auto Scale │  │- Spam       │  │- Dashboards │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           DATABASE LAYER                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │PostgreSQL   │  │Cassandra    │  │MongoDB      │  │ClickHouse   │            │
│  │Cluster      │  │Cluster      │  │Cluster      │  │Cluster      │            │
│  │- Users      │  │- Messages   │  │- Groups     │  │- Analytics  │            │
│  │- Groups     │  │- Chat Hist  │  │- Metadata   │  │- Events     │            │
│  │- Metadata   │  │- Media Meta │  │- Contacts   │  │- Metrics    │            │
│  │- Config     │  │- Call Logs  │  │- Settings   │  │- Logs       │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
│                                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Object       │  │Search Engine│  │Time Series  │  │Graph DB     │            │
│  │Storage      │  │(Elasticsearch)│ │(InfluxDB)   │  │(Neo4j)      │            │
│  │- Media Files│  │- Chat Search│  │- Metrics    │  │- Social     │            │
│  │- Backups    │  │- User Search│  │- Monitoring │  │- Network    │            │
│  │- Archives   │  │- Content    │  │- Analytics  │  │- Contacts   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘            │
└─────────────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      EXTERNAL SERVICES & INTEGRATIONS                          │
├─────────────────────────────────────────────────────────────────────────────────┤
│   [FCM/APNs Push]  [SMS Gateway]  [Email Service]   [Payment Gateway]         │
│   [Voice/Video Infrastructure]    [Content Moderation]    [ML/AI Services]    │
│   [Telecom APIs]   [Cloud Storage]   [CDN Providers]   [Security Services]    │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                        MONITORING & OBSERVABILITY                              │
├─────────────────────────────────────────────────────────────────────────────────┤
│  [Prometheus/Grafana] [ELK Stack] [Jaeger Tracing] [PagerDuty] [Custom Tools] │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 4. Component Functionality Explanation

### CLIENT LAYER Components
- **iOS/Android Apps**: Native mobile applications with offline capabilities
- **Web App**: Browser-based WhatsApp Web interface
- **Desktop App**: Native desktop applications for Windows/Mac/Linux
- **All clients maintain local SQLite databases for offline message storage**

### EDGE LAYER Components
- **Global CDN**: Distributes media content globally for fast access
- **Edge Servers**: Regional servers that cache content and provide geo-routing
- **Reduces latency by serving content from nearest location**

### LOAD BALANCER Components
- **L4 Load Balancer**: Handles WebSocket connections at transport layer
- **L7 Load Balancer**: Routes HTTP/API requests at application layer
- **Connection Balancing**: Distributes persistent connections across servers

### API GATEWAY Components
- **Authentication**: Validates JWT tokens and user permissions
- **Rate Limiting**: Prevents abuse by limiting requests per user
- **Protocol Translation**: Converts between different protocols (HTTP, WebSocket)
- **Request Routing**: Directs requests to appropriate microservices

### CONNECTION MANAGEMENT LAYER Components

#### WebSocket Manager
- **Maintains**: 1 billion+ concurrent WebSocket connections
- **Routes**: Messages to correct user connections
- **Heartbeat**: Keeps connections alive and detects disconnections
- **Load Balancing**: Distributes connections across multiple servers

#### Connection Pool
- **Capacity**: 1 million connections per server
- **Scaling**: Auto-scales based on connection count
- **Failover**: Handles server failures gracefully

#### Presence Service
- **Online Status**: Tracks which users are currently online
- **Last Seen**: Records when users were last active
- **Typing Indicators**: Shows when users are typing messages

#### Session Manager
- **User Sessions**: Manages active user sessions across devices
- **Multi-Device**: Synchronizes messages across multiple devices
- **Security**: Handles session timeouts and validation

### CORE SERVICES LAYER Components

#### User Service
- **Registration**: Phone number verification with OTP
- **Authentication**: JWT token generation and validation
- **Profile Management**: Name, photo, status, privacy settings
- **Contact Management**: Phone book sync and contact discovery

#### Message Service
- **Send/Receive**: Core messaging functionality
- **Message History**: Store and retrieve chat history
- **End-to-End Encryption**: Signal Protocol implementation
- **Message Status**: Sent, delivered, read receipts tracking

#### Group Service
- **Group Creation**: Create new groups with members
- **Member Management**: Add/remove members, admin controls
- **Group Settings**: Name, description, privacy settings
- **Broadcast Lists**: Send messages to multiple contacts

#### Media Service
- **Upload Processing**: Handle image, video, document uploads
- **Compression**: Reduce file sizes for efficient transfer
- **Thumbnail Generation**: Create preview images
- **Format Conversion**: Convert media to supported formats

#### Call Service
- **Voice Calls**: One-on-one and group voice calling
- **Video Calls**: Video conferencing with multiple participants
- **Call Routing**: Find optimal path between users
- **Quality Optimization**: Adaptive bitrate based on connection

#### Notification Service
- **Push Notifications**: FCM for Android, APNs for iOS
- **WebSocket Notifications**: Real-time in-app notifications
- **Email Notifications**: Backup notification method
- **Smart Delivery**: Choose best notification method

#### Analytics Service
- **Usage Metrics**: Message volume, user engagement
- **Performance Monitoring**: Service health and response times
- **ML Models**: Improve features and detect patterns
- **Business Intelligence**: Insights for product decisions

#### Security Service
- **Encryption Management**: Key generation and rotation
- **Spam Detection**: Identify and block spam messages
- **Content Moderation**: Detect inappropriate content
- **Fraud Prevention**: Protect against abuse and fraud

## 5. End-to-End Component Flows

### Flow 1: User Registration & Authentication

#### Phone Number Registration Flow
```
User enters phone number → Mobile App → API Gateway → User Service
    ↓
User Service validates phone number format and country
    ↓
User Service → SMS Gateway (send OTP code)
    ↓
User Service → Redis (store OTP with 5-minute TTL)
    ↓
User enters OTP → User Service verifies OTP
    ↓
User Service → PostgreSQL (create user record)
    ↓
User Service → JWT Service (generate access token)
    ↓
User Service → Redis (cache user session)
    ↓
Return JWT token and user profile to client
    ↓
Client stores token for subsequent API calls
```

#### Multi-Device Authentication Flow
```
User opens WhatsApp Web → Scan QR Code
    ↓
Web Client generates QR code with temporary session ID
    ↓
Mobile App scans QR code → Extract session ID
    ↓
Mobile App → User Service (validate user + session ID)
    ↓
User Service → Session Manager (create web session)
    ↓
User Service → WebSocket Manager (establish web connection)
    ↓
Web Client receives authentication confirmation
    ↓
Sync chat history and contacts to web client
```

### Flow 2: Real-time Message Delivery

#### One-to-One Message Flow
```
User A types message → Mobile App (local SQLite storage)
    ↓
App → WebSocket Connection → Connection Manager
    ↓
Connection Manager → Message Service (validate and process)
    ↓
Message Service → End-to-End Encryption (encrypt with User B's public key)
    ↓
Message Service → Cassandra (store encrypted message)
    ↓
Message Service → Kafka (publish message_sent event)
    ↓
Message Delivery Service consumes event:
    ↓
    Check if User B is online:
        If ONLINE: WebSocket Manager → Direct delivery to User B
        If OFFLINE: Notification Service → Push notification
    ↓
User B receives message → Send delivery receipt
    ↓
Delivery receipt → Message Service → Update message status
    ↓
Message Service → WebSocket → User A (delivery confirmation)
```

#### Message Status Updates Flow
```
Message Delivery Process:
1. SENT: Message stored in sender's device and server
2. DELIVERED: Message delivered to recipient's device
3. READ: Recipient opened and read the message

Status Update Flow:
User B opens chat → Mobile App → Message Service
    ↓
Message Service → Cassandra (mark messages as read)
    ↓
Message Service → WebSocket → User A (read receipt)
    ↓
User A's app shows blue checkmarks (read status)
```

### Flow 3: Group Messaging

#### Group Message Distribution Flow
```
User A sends message to Group (100 members) → Message Service
    ↓
Message Service → Group Service (get group member list)
    ↓
Message Service → Encrypt message with group key
    ↓
Message Service → Cassandra (store group message)
    ↓
Message Service → Kafka (publish group_message event)
    ↓
Group Message Distributor consumes event:
    ↓
    For each group member:
        Check online status
        If ONLINE: WebSocket delivery
        If OFFLINE: Queue for push notification
    ↓
Batch process delivery to optimize performance
    ↓
Collect delivery receipts from all members
    ↓
Update message status based on delivery success rate
```

#### Group Management Flow
```
User creates new group → Group Service
    ↓
Group Service → PostgreSQL (create group record)
    ↓
Group Service → Add creator as admin
    ↓
User adds members → Group Service validates phone numbers
    ↓
Group Service → Send group invitations
    ↓
Group Service → Kafka (publish group_created event)
    ↓
Analytics Service tracks group creation metrics
    ↓
Notification Service sends "Added to group" notifications
```

### Flow 4: Media Sharing

#### Media Upload Flow
```
User selects photo → Mobile App compresses locally
    ↓
App → Media Service (upload original + compressed versions)
    ↓
Media Service → Object Storage S3 (store files)
    ↓
Media Service → Generate unique media ID and URLs
    ↓
Media Service → Kafka (publish media_uploaded event)
    ↓
Background Media Processor:
    - Generate thumbnails (150x150, 300x300)
    - Create compressed versions for different devices
    - Extract metadata (EXIF data, duration, etc.)
    - Virus scanning and content moderation
    ↓
Media Service → CDN (distribute processed media globally)
    ↓
Return media URLs to client for message attachment
```

#### Media Download Flow
```
User opens chat with media → Mobile App requests media
    ↓
App → CDN (check for cached media)
    ↓
CDN HIT: Serve from nearest edge location
CDN MISS: CDN → Origin S3 → Cache at edge → Serve to user
    ↓
Adaptive delivery based on:
    - Network speed (WiFi vs cellular)
    - Device capabilities (screen resolution)
    - User preferences (auto-download settings)
```

### Flow 5: Voice/Video Calling

#### Call Initiation Flow
```
User A initiates call to User B → Call Service
    ↓
Call Service → Check User B availability and permissions
    ↓
Call Service → WebRTC Signaling Server (establish peer connection)
    ↓
Call Service → STUN/TURN servers (NAT traversal)
    ↓
If direct connection fails: Route through media relay servers
    ↓
Call Service → Notification Service (ring User B)
    ↓
User B accepts → Establish media stream (P2P when possible)
    ↓
Call Service → Monitor call quality and adjust bitrate
    ↓
Call ends → Call Service → Analytics (call duration, quality metrics)
```

#### Group Video Call Flow
```
User creates group call → Call Service
    ↓
Call Service → Media Server (allocate conferencing resources)
    ↓
Call Service → Invite group members simultaneously
    ↓
Each participant connects to media server
    ↓
Media Server:
    - Mixes audio streams
    - Handles video layout (grid view, speaker view)
    - Adapts quality based on participant bandwidth
    ↓
Real-time quality monitoring and auto-scaling
    ↓
Call analytics and billing (for business features)
```

### Flow 6: Online Presence & Last Seen

#### Presence Management Flow
```
User opens WhatsApp → WebSocket connection established
    ↓
Connection Manager → Presence Service (mark user online)
    ↓
Presence Service → Redis (update online status with TTL)
    ↓
Presence Service → Broadcast online status to contacts
    ↓
User closes app → WebSocket disconnection detected
    ↓
Presence Service → Update last seen timestamp
    ↓
Presence Service → Redis (remove from online users)
    ↓
Background job updates "last seen" for privacy settings
```

#### Typing Indicators Flow
```
User starts typing → Mobile App detects keystroke
    ↓
App → WebSocket → Presence Service (typing_start event)
    ↓
Presence Service → Check privacy settings
    ↓
If allowed: Broadcast typing indicator to chat participants
    ↓
Typing stops or 10 seconds timeout → typing_stop event
    ↓
Presence Service → Clear typing indicators
```

### Flow 7: Message Search & History

#### Chat Search Flow
```
User searches "restaurant" → Mobile App checks local SQLite
    ↓
Local results returned immediately for recent chats
    ↓
App → Search Service (for comprehensive server search)
    ↓
Search Service → Elasticsearch (query encrypted message index)
    ↓
Search Service → Decrypt relevant results for user
    ↓
Search Service → Rank results by relevance and recency
    ↓
Return paginated search results to client
    ↓
Analytics Service logs search patterns (anonymized)
```

#### Chat History Sync Flow
```
New device setup → User authentication complete
    ↓
App → Message Service (request chat history sync)
    ↓
Message Service → Cassandra (fetch encrypted message history)
    ↓
Message Service → Decrypt messages with user's private key
    ↓
Batch transfer chat history (newest first)
    ↓
App → Local SQLite (store decrypted messages)
    ↓
Progress indicator shows sync status to user
    ↓
Background sync continues for older messages
```

### Flow 8: Push Notifications

#### Offline Message Notification Flow
```
Message delivery attempt → User offline detected
    ↓
Message Service → Notification Queue (queue notification)
    ↓
Notification Service processes queue:
    ↓
    Check user notification preferences
    Check do-not-disturb settings
    Check message priority (normal vs urgent)
    ↓
Notification Service → FCM/APNs (send push notification)
    ↓
Mobile OS delivers notification to device
    ↓
User taps notification → App opens → Fetch latest messages
    ↓
Mark notification as delivered and clear from queue
```

#### Smart Notification Optimization
```
Notification Service uses ML models to optimize:
    ↓
    Batch notifications from same sender
    Suppress notifications for active conversations
    Adjust timing based on user timezone and habits
    Prioritize important contacts and keywords
    ↓
A/B testing for notification effectiveness
    ↓
Analytics feedback loop improves notification algorithms
```

### Flow 9: End-to-End Encryption

#### Encryption Key Management Flow
```
User registration → Generate key pair (public/private keys)
    ↓
User Service → Store public key on server
    ↓
Private key stored locally on device (never leaves device)
    ↓
Message encryption process:
        Sender → Generate session key for conversation
        Encrypt message with session key
        Encrypt session key with recipient's public key
        Send encrypted message + encrypted session key
    ↓
Recipient decryption:
        Decrypt session key with private key
        Decrypt message with session key
        Display plaintext message
```

#### Key Rotation and Security
```
Periodic key rotation (every 30 days):
    ↓
    Generate new key pair
    Notify contacts of new public key
    Maintain old keys for message history decryption
    ↓
Security verification:
    Users can verify identity through QR codes
    Compare security codes to detect man-in-middle attacks
    ↓
Forward secrecy: Old messages remain secure even if keys compromised
```

### Flow 10: System Monitoring & Health

#### Real-time Monitoring Flow
```
All services → Metrics collectors (Prometheus)
    ↓
Time-series database stores metrics:
    - Message delivery rate and latency
    - WebSocket connection count
    - Database response times
    - Error rates and failure modes
    ↓
Grafana dashboards visualize system health
    ↓
Alert Manager monitors critical thresholds:
    - Message delivery failure > 0.1%
    - Average latency > 100ms
    - WebSocket disconnection rate > 5%
    ↓
PagerDuty alerts on-call engineers for critical issues
```

#### Auto-scaling and Load Management
```
Load Balancer monitors server metrics
    ↓
High CPU/Memory usage detected → Auto-scaling triggered
    ↓
Kubernetes/ECS spins up new service instances
    ↓
Load balancer adds new instances to rotation
    ↓
Connection Manager redistributes WebSocket connections
    ↓
Gradual traffic shift to new instances
    ↓
Monitor stability before completing scale-up operation
```

## 6. Performance Characteristics

### Message Delivery Performance
- **One-to-One Messages**: < 100ms end-to-end delivery
- **Group Messages**: < 500ms for groups up to 256 members
- **Media Messages**: 2-10 seconds depending on file size
- **Voice Messages**: < 200ms for playback start

### System Throughput
- **Peak Messages
