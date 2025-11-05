# System Design & Scalability - Beginner Level Answers

---

## 1. What is horizontal vs vertical scaling?

**Vertical scaling (Scale up):**
- Add more power to existing machine
- More CPU, RAM, storage
- Simpler (no code changes)
- Limited by hardware
- Single point of failure

**Horizontal scaling (Scale out):**
- Add more machines
- Distribute load
- Better availability
- Unlimited scaling potential
- Requires stateless design

**Best practice:** Design for horizontal scaling from start.

---

## 2. Explain what load balancing is

**Load balancing:** Distributing incoming traffic across multiple servers.

**Purpose:**
- Prevent single server overload
- Improve availability
- Enable horizontal scaling
- Better performance

**Algorithms:**
- Round robin (sequential)
- Least connections
- IP hash (sticky sessions)
- Weighted (based on capacity)

**Types:** Hardware, software (Nginx, HAProxy), cloud (AWS ELB)

---

## 3. What is caching and why is it important?

**Caching:** Store frequently accessed data in fast storage for quick retrieval.

**Benefits:**
- Faster response times
- Reduced database load
- Lower latency
- Cost savings

**Where to cache:**
- Browser (HTTP caching)
- CDN (static files)
- Application (Redis, Memcached)
- Database (query cache)

**Challenge:** Cache invalidation (when to update/remove)

---

## 4. What is the difference between stateful and stateless applications?

**Stateless:**
- No session data stored on server
- Each request independent
- Can route to any server
- Easy to scale
- Example: RESTful APIs

**Stateful:**
- Server remembers client state
- Session data on server
- Requests must go to same server
- Harder to scale
- Example: WebSocket connections

**Best practice:** Stateless design for scalability, store state in database/Redis.

---

## 5. Explain what an API gateway is

**API Gateway:** Single entry point for all client requests to backend services.

**Responsibilities:**
- Routing to appropriate service
- Authentication/authorization
- Rate limiting
- Request/response transformation
- Logging and monitoring
- SSL termination

**Benefits:**
- Single public endpoint
- Centralized concerns
- Hide internal structure
- Protocol translation

**Examples:** Kong, AWS API Gateway, Nginx.

---

## 6. What is rate limiting?

**Rate limiting:** Restricting number of requests a client can make in time period.

**Purpose:**
- Prevent abuse
- Protect from DDoS
- Ensure fair usage
- Control costs

**Implementation:**
- Fixed window (X requests per minute)
- Sliding window (more accurate)
- Token bucket (smooth rate)

**Response:** 429 Too Many Requests with Retry-After header.

---

## 7. What is the purpose of a reverse proxy?

**Reverse proxy:** Server that sits in front of web servers, forwarding client requests.

**Benefits:**
- Load balancing
- SSL termination
- Caching
- Compression
- Security (hide backend)
- Static file serving

**vs Forward proxy:** Reverse = protects servers, Forward = protects clients

**Example:** Nginx as reverse proxy in front of Node.js apps.

---

## 8. Explain what CDN is and its benefits

**CDN (Content Delivery Network):** Distributed network of servers caching content close to users.

**How it works:**
- Content cached in multiple locations
- User gets content from nearest server
- Reduces latency

**Benefits:**
- Faster load times
- Reduced server load
- Better availability
- Handle traffic spikes
- DDoS protection

**Use for:** Images, CSS, JavaScript, videos, static files.

**Examples:** Cloudflare, AWS CloudFront, Akamai.

---

## 9. What are webhooks?

**Webhooks:** HTTP callbacks that notify your application when events occur in external system.

**How it works:**
1. Register URL with service
2. Event occurs
3. Service POSTs data to your URL
4. Your app processes event

**vs Polling:** Push vs pull, more efficient

**Use cases:**
- Payment confirmations
- GitHub events
- Slack notifications

**Implementation:** Verify signatures, handle retries, idempotent processing.

---

## 10. What is the difference between authentication and authorization?

**Authentication (AuthN):** Verify who you are
- Login process
- Username/password, token, biometrics
- Proves identity

**Authorization (AuthZ):** Verify what you can do
- Permissions and access control
- Comes after authentication
- Role-based, resource-based

**Example:**
- Authentication: Login as user123
- Authorization: Check if user123 can edit post456

---

## 11. Explain what session management is

**Session management:** Track user state across multiple requests.

**Methods:**

**1. Session cookies:**
- Server stores session data
- Cookie contains session ID
- Traditional approach

**2. JWT tokens:**
- Self-contained tokens
- Stateless
- Client stores token

**Security:**
- Use HTTPS
- Secure, HttpOnly cookies
- Session expiration
- CSRF protection

---

## 12. What is the purpose of logging?

**Logging:** Recording application events and errors.

**Purposes:**
- Debugging issues
- Monitoring health
- Audit trail
- Performance analysis
- Security incidents

**Log levels:**
- ERROR: Errors requiring attention
- WARN: Potential problems
- INFO: Informational messages
- DEBUG: Detailed debugging

**Best practices:**
- Structured logging (JSON)
- Don't log sensitive data
- Appropriate levels
- Centralized logging

---

## 13. What are health checks in web applications?

**Health checks:** Endpoints that report application health status.

**Types:**

**Liveness:**
- Is app running?
- Restart if failing

**Readiness:**
- Is app ready to serve?
- Remove from load balancer if not

**Typical checks:**
- Database connectivity
- Dependent service availability
- Disk space
- Memory usage

**Response:** 200 OK if healthy, 503 if not

**Used by:** Load balancers, Kubernetes, monitoring

---

## 14. Explain what a message queue is

**Message queue:** Asynchronous communication between services using messages.

**How it works:**
- Producer sends messages to queue
- Consumer receives and processes
- Decoupled communication

**Benefits:**
- Decouple services
- Handle load spikes (buffer)
- Retry failed messages
- Guaranteed delivery
- Async processing

**Use cases:**
- Email sending
- Image processing
- Background jobs
- Event-driven architecture

**Examples:** RabbitMQ, Redis, AWS SQS, Kafka.

---

## 15. What is the difference between SQL and NoSQL databases?

**SQL (Relational):**
- Structured data, fixed schema
- Tables with relations (joins)
- ACID transactions
- Examples: PostgreSQL, MySQL

**NoSQL:**
- Flexible schema
- Various types (document, key-value, graph)
- Horizontal scaling
- Examples: MongoDB, Redis, Cassandra

**When to use SQL:**
- Complex queries
- Strict consistency
- Relationships important

**When to use NoSQL:**
- Flexible schema
- Rapid development
- Horizontal scaling
- Large data volumes
