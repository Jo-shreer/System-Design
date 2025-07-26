# WhatsApp Comprehensive End-to-End Flow - All Cases

## 1. User Registration & Onboarding Flow

### 1.1 First Time User Registration
```
[User Downloads WhatsApp] 
    ↓
[Opens App → Enters Phone Number]
    ↓
[Client Layer] → [Edge Server] → [L7 Load Balancer] → [API Gateway]
    ↓
[API Gateway] → [User Service] → [Phone Number Validation]
    ↓
[User Service] → [SMS Gateway Integration] → [External SMS Service]
    ↓
[SMS Service] → [User's Phone (Receives OTP)]
    ↓
[User Enters OTP] → [Client] → [API Gateway] → [User Service]
    ↓
[User Service] → [PostgreSQL] (Creates user record)
    ↓
[User Service] → [Generate Signal Protocol Keys] → [Store Public Key]
    ↓
[Redis Cache] (Store session) → [Return Success to Client]
```

**Detailed Process:**
1. **App Installation**: User downloads from App Store/Play Store
2. **Phone Number Entry**: Client validates format locally first
3. **Server Validation**: User Service checks if number is valid and not banned
4. **Rate Limiting**: API Gateway checks if too many attempts from this IP/device
5. **SMS Generation**: Creates random 6-digit OTP with 5-minute expiry
6. **SMS Delivery**: Routes through multiple SMS providers for reliability
7. **OTP Verification**: User has 3 attempts, after which new OTP needed
8. **Key Generation**: Client generates Signal Protocol key pairs locally
9. **Profile Creation**: Creates user profile with basic metadata
10. **Session Creation**: Generates JWT token and stores in Redis

### 1.2 Contact Discovery Process
```
[User Grants Contact Permission]
    ↓
[Client] → [Hash Phone Numbers Locally] → [Upload Hashed Numbers]
    ↓
[API Gateway] → [User Service] → [Contact Matching Service]
    ↓
[Contact Service] → [PostgreSQL] (Query existing users)
    ↓
[Return Matched Contacts] → [Client Updates Contact List]
    ↓
[Client] → [Download Profile Pictures] → [CDN] → [Cache Locally]
```

## 2. Message Sending Flow - All Scenarios

### 2.1 Simple Text Message (Online Recipient)
```
[User Types "Hello" in Chat]
    ↓
[Client Layer Processing:]
    - Check internet connectivity
    - Validate message length (<65KB)
    - Generate message ID (UUID)
    - Check Signal Protocol session exists
    ↓
[Encryption Process:]
    - Generate random AES-256 key for this message
    - Encrypt "Hello" with AES key
    - Encrypt AES key with recipient's public key (Signal Protocol)
    - Create message envelope with metadata
    ↓
[Network Transmission:]
[iOS/Android App] → [WebSocket Connection] → [Nearest Edge Server]
    ↓
[Edge Server Processing:]
    - SSL termination and decryption of transport layer
    - Route determination based on recipient's location
    - Traffic type identification (real-time message)
    ↓
[L4 Load Balancer] (WebSocket Traffic)
    - Connection pooling and health check
    - Route to least loaded WebSocket server
    - Maintain connection state
    ↓
[API Gateway Processing:]
    - JWT token validation
    - Rate limiting check (100 messages/minute per user)
    - Protocol translation (WebSocket → Internal RPC)
    - Spam detection (basic content filtering)
    - Route to Message Service
    ↓
[Connection Management Layer:]
[WebSoc
