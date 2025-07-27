# Most Popular System Design Interview Questions - Complete List

## Messaging and Communication Systems

### WhatsApp / Telegram / Signal
Design a messaging system that supports: Real-time messaging between users, Group chats with 1000+ members, Media sharing (images, videos, documents), End-to-end encryption, Online/offline status and last seen, Message delivery status (sent, delivered, read), Push notifications for offline users, Multi-device synchronization.

Key Focus Areas: WebSocket connections for real-time messaging, Message queuing and delivery guarantees, Database design for chat history, Caching strategies for active conversations, Push notification systems, Load balancing for millions of concurrent users.

### Slack / Discord / Microsoft Teams
Design a team communication platform with: Channels and direct messages, File sharing and integrations, Video/voice calling, Message threading and reactions, Search functionality across conversations, Bot integrations and webhooks, User presence and status updates.

Key Focus Areas: Channel-based messaging architecture, File storage and CDN for media, Search indexing for message history, Real-time collaboration features, Integration with third-party services.

### Zoom / Google Meet / Video Conferencing
Design a video conferencing system supporting: HD video and audio streaming, Screen sharing capabilities, Recording and playback, Chat during meetings, Waiting rooms and meeting controls, Integration with calendar systems, Mobile and web client support.

Key Focus Areas: Real-time video/audio streaming protocols, Media servers and bandwidth optimization, Recording storage and processing, Signaling servers for connection establishment, Geographic distribution for low latency.

## Social Media Platforms

### Twitter / X
Design a microblogging platform with: Tweet creation and timeline display, Following/follower relationships, Real-time feed updates, Trending topics and hashtags, Search functionality, Media attachments, Retweets and replies, Direct messaging.

Key Focus Areas: Timeline generation algorithms, Fan-out strategies for tweet delivery, Trending topic calculation, Search indexing and ranking, Celebrity user scalability (millions of followers), Real-time notifications.

### Instagram / TikTok
Design a photo/video sharing platform with: Media upload and processing, User feeds and discovery, Stories/temporary content, Direct messaging, Hashtags and search, User profiles and followers, Recommendations and explore page.

Key Focus Areas: Image/video processing pipelines, CDN for global media delivery, Recommendation algorithms, Feed generation and ranking, Mobile-first architecture, Storage optimization for media files.

### Facebook / Social Network
Design a comprehensive social network with: News feed and post creation, Friend connections and suggestions, Groups and pages, Events and notifications, Marketplace functionality, Real-time chat, Photo/video sharing, Privacy controls.

Key Focus Areas: Complex relationship modeling, News feed ranking algorithms, Privacy and security at scale, Real-time notifications, Content moderation, Graph database for social connections.

## E-commerce and Marketplace

### Amazon / E-commerce Platform
Design an online marketplace with: Product catalog and search, Shopping cart and checkout, Payment processing, Order management, Inventory tracking, Seller dashboard, Reviews and ratings, Recommendation engine.

Key Focus Areas: Product search and filtering, Inventory consistency across sellers, Payment processing security, Order fulfillment workflows, Recommendation algorithms, Scalable product catalog, Price and inventory updates.

### Uber / Lyft / Ride Sharing
Design a ride-sharing platform with: Driver and rider matching, Real-time location tracking, Route optimization, Dynamic pricing (surge pricing), Payment processing, Trip history and ratings, Driver earnings tracking.

Key Focus Areas: Geospatial indexing and location services, Real-time matching algorithms, Pricing calculation engines, GPS tracking and route optimization, Payment processing, Driver-rider communication.

### Airbnb / Booking Platform
Design a vacation rental platform with: Property listings and search, Booking and availability management, Payment processing, Host and guest profiles, Reviews and ratings, Messaging between hosts/guests, Calendar management.

Key Focus Areas: Search with geographic and filter criteria, Availability calendar management, Payment escrow systems, Image storage and optimization, Booking conflict resolution, Trust and safety systems.

## Content and Media

### YouTube / Video Streaming
Design a video sharing platform with: Video upload and processing, Video streaming and playback, Search and recommendations, User channels and subscriptions, Comments and interactions, Analytics for creators, Live streaming capabilities.

Key Focus Areas: Video transcoding and storage, CDN for global video delivery, Recommendation algorithms, Search indexing for video content, Real-time analytics, Live streaming infrastructure, Content moderation.

### Netflix / Video Streaming Service
Design a video streaming service with: Content catalog and metadata, Video streaming with adaptive bitrate, User profiles and preferences, Recommendation system, Download for offline viewing, Content licensing and DRM, Global content delivery.

Key Focus Areas: Adaptive bitrate streaming, Content recommendation algorithms, Global CDN architecture, DRM and content protection, Personalization and user profiling, Analytics and viewing patterns.

### Spotify / Music Streaming
Design a music streaming platform with: Music catalog and metadata, Audio streaming, Playlist creation and sharing, Music recommendations, Social features (following artists/friends), Offline music download, Artist analytics dashboard.

Key Focus Areas: Music recommendation algorithms, Audio streaming optimization, Playlist management, Social music discovery, Licensing and royalty tracking, Mobile-optimized streaming.

## Collaboration and Productivity

### Google Drive / Dropbox / Cloud Storage
Design a file storage and sharing service with: File upload and synchronization, Folder sharing and permissions, Version control, Real-time collaboration, Search functionality, Mobile and desktop apps, Backup and recovery.

Key Focus Areas: File synchronization algorithms, Conflict resolution for concurrent edits, Permission management, Storage optimization and deduplication, Real-time collaboration protocols, Cross-platform client applications.

### Google Docs / Collaborative Editor
Design a real-time collaborative document editor with: Multiple user editing, Conflict resolution, Version history, Commenting and suggestions, Document sharing and permissions, Real-time cursor tracking, Export to various formats.

Key Focus Areas: Operational transformation for concurrent editing, Real-time synchronization, Conflict resolution algorithms, Document versioning, Collaborative features like comments, Permission and access control.

### Trello / Jira / Project Management
Design a project management tool with: Board and card management, Team collaboration, Task assignment and tracking, Due dates and notifications, File attachments, Activity feeds, Reporting and analytics.

Key Focus Areas: Flexible data modeling for projects, Real-time collaboration updates, Notification systems, Search and filtering, Integration with other tools, Permission management for teams.

## Search and Discovery

### Google Search Engine
Design a web search engine with: Web crawling and indexing, Search query processing, Ranking algorithms, Auto-complete suggestions, Image and video search, Personalized results, Ad placement, Spam detection.

Key Focus Areas: Distributed web crawling, Inverted index construction, PageRank and ranking algorithms, Query processing optimization, Index sharding and replication, Real-time search suggestions.

### Elasticsearch / Search Service
Design a distributed search service with: Document indexing, Full-text search, Faceted search and filters, Auto-complete, Search analytics, Multi-language support, Real-time indexing updates.

Key Focus Areas: Distributed indexing, Query optimization, Relevance scoring, Index partitioning, Real-time updates, Search result aggregation, Performance optimization.

## Financial and Payment Systems

### PayPal / Venmo / Payment System
Design a digital payment platform with: User account management, Money transfers, Payment processing, Transaction history, Fraud detection, Multi-currency support, Merchant integration, Dispute resolution.

Key Focus Areas: Transaction processing at scale, Fraud detection algorithms, Regulatory compliance, Double-entry bookkeeping, Payment gateway integration, Security and encryption, Real-time balance updates.

### Stock Trading Platform
Design a stock trading system with: Real-time stock prices, Order placement and matching, Portfolio management, Market data feeds, Trading history, Risk management, Regulatory compliance.

Key Focus Areas: Order matching engines, Real-time price feeds, Transaction processing, Risk management systems, Market data distribution, Regulatory reporting, High-frequency trading support.

## Infrastructure and Platform Services

### URL Shortener (bit.ly / tinyurl)
Design a URL shortening service with: URL encoding and decoding, Custom short URLs, Click tracking and analytics, Expiration dates, Bulk URL operations, API for developers, Spam protection.

Key Focus Areas: URL encoding algorithms, Database design for billions of URLs, Caching strategies, Analytics data collection, Rate limiting, Geographic distribution, Custom domain support.

### Pastebin / Code Sharing
Design a text sharing service with: Text/code snippet storage, Syntax highlighting, Expiration options, Privacy controls, Search functionality, User accounts, API access.

Key Focus Areas: Text storage optimization, Search indexing, Expiration management, Syntax highlighting, Access control, API rate limiting, Content moderation.

### Design a Cache System (Redis/Memcached)
Design a distributed caching system with: Key-value storage, TTL (time-to-live) support, Distributed cache nodes, Cache eviction policies, Replication and failover, Memory optimization, Client SDKs.

Key Focus Areas: Distributed hashing, Memory management, Eviction algorithms (LRU, LFU), Replication strategies, Network protocols, Performance optimization, Monitoring and metrics.

## Analytics and Monitoring

### Google Analytics / Web Analytics
Design a web analytics platform with: Page view tracking, User behavior analysis, Real-time dashboard, Custom event tracking, Conversion funnel analysis, A/B testing support, Data export capabilities.

Key Focus Areas: Event collection at scale, Real-time data processing, Data aggregation and reporting, Time-series data storage, Dashboard generation, Data privacy compliance.

### Monitoring System (DataDog/NewRelic)
Design an application monitoring system with: Metrics collection, Log aggregation, Alert management, Dashboard creation, Distributed tracing, Performance monitoring, Incident response.

Key Focus Areas: Time-series database design, Alert processing, Log indexing and search, Visualization and dashboards, Agent deployment, Data retention policies.

## Gaming and Real-time Systems

### Online Gaming Platform
Design a multiplayer gaming platform with: Real-time game state synchronization, Matchmaking, Leaderboards, In-game chat, Virtual currency, Tournament system, Anti-cheat mechanisms.

Key Focus Areas: Real-time synchronization, Game state management, Matchmaking algorithms, Cheat detection, Virtual economy, Low-latency networking, Global server distribution.

### Live Streaming (Twitch)
Design a live streaming platform with: Video ingestion and transcoding, Real-time chat, Stream discovery, Follower notifications, VOD (video on demand), Monetization features, Content moderation.

Key Focus Areas: Video streaming protocols, Real-time chat at scale, Content delivery networks, Stream processing, Recommendation algorithms, Content moderation tools.

## Common Interview Tips

### Approach Strategy
Start with requirements gathering, Ask clarifying questions about scale and features, Draw high-level architecture first, Deep dive into specific components, Discuss trade-offs and alternatives, Address scalability and reliability concerns, Consider monitoring and deployment.

### Key Areas to Cover
Scalability: How to handle growth from 1K to 100M users, Database design: SQL vs NoSQL, sharding, replication, Caching: Where and what to cache, Load balancing: Layer 4 vs Layer 7, CDN usage, Microservices vs monolith trade-offs, Security considerations, Monitoring and observability.

### Scale Numbers to Remember
Daily Active Users: 1K (small), 1M (medium), 100M+ (large scale), Read/Write Ratio: Usually 100:1 or 1000:1 reads to writes, Database QPS: MySQL ~1K QPS, Cassandra ~10K QPS per node, Cache Hit Ratio: Aim for 80-95% cache hit rates, Response Time: <100ms for good user experience, Storage: 1KB per message, 1MB per photo, 10MB per video.

The key to succeeding in system design interviews is practicing these problems, understanding the trade-offs between different architectural choices, and being able to explain your reasoning clearly while designing for the specific scale and requirements given.
