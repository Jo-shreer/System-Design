What is a URL shortening service?
A URL shortening service is a system that takes a long web address (URL) and converts it into a much shorter, easy-to-share link.

For example:
https://www.example.com/articles/how-to-design-a-scalable-system
becomes
https://short.ly/abc123
When someone clicks or types the short URL, they get redirected to the original long URL.

Why do we need it?
Convenience: Short links are easier to share on social media, in texts, or printed material.
Tracking: It allows tracking clicks and analytics on the short URL.
Aesthetics: Looks cleaner, especially in character-limited platforms like Twitter.

What does designing such a service involve?
Generating a unique short code:
Every long URL gets mapped to a unique short code (like abc123). The system must ensure no two long URLs share the same short code.

Storing the mapping:
The service stores the association between the short code and the original long URL.

Redirecting:
When users visit the short URL, the service redirects them to the original long URL.

Scalability & Performance:
Since many users click short URLs, the system must handle high read traffic (redirects) efficiently.

Reliability:
The system should be highly available — short URLs should always work, even if some servers fail.

Additional features (optional):
Analytics on clicks (where/when)
Custom short URLs (like short.ly/mybrand)
Expiration of links

*************************************************************************************************************************

1. Generating unique short URLs
Use a base62 encoding (letters + digits) to convert an auto-incrementing ID or hash into a short string.
Alternatively, generate a random string of fixed length and check for collisions.
Use a centralized service or a distributed ID generator (like Twitter’s Snowflake) to generate unique IDs without conflicts.

2. Handling high read/write traffic
Write traffic:
Writes are relatively low volume; use a distributed database like Cassandra or DynamoDB to store mappings (short URL → long URL).

Read traffic:
Reads are very high (redirects). Use caching (e.g., Redis or CDN edge caches) to serve redirects quickly.
Use load balancers and horizontal scaling for the web servers.

3. Ensuring availability and avoiding collisions
Replicate databases across regions for fault tolerance.
Use consistent hashing to shard the data and distribute load.
Implement retries and fallbacks in case of service failures.

For collisions, check if a generated short URL already exists before committing; regenerate if needed.

