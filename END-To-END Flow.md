End-to-End Component Flows
Flow 1: User Registration & Authentication Module
User Registration Flow

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
