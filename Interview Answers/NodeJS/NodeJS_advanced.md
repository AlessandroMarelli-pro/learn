# Node.js - Advanced Level Answers

---

## 1. How would you architect a Node.js application for horizontal scaling?

**Key principles:**

**1. Stateless design:**
- No local session storage
- Store state in Redis/database
- Each request independent

**2. Shared services:**
- Database connection pooling
- Centralized caching (Redis)
- Message queues for jobs

**3. Load balancing:**
- Nginx, HAProxy, or cloud load balancer
- Round-robin or least connections
- Health check endpoints

**4. Multi-instance deployment:**
- Docker containers
- Kubernetes orchestration
- PM2 cluster mode

**5. Async communication:**
- Message queues (RabbitMQ, Redis)
- Event-driven architecture
- Decouple services

**6. Monitoring:**
- Distributed tracing
- Centralized logging
- Performance metrics

---

## 2. Explain the internals of Node.js event loop in detail (phases, queues)

**Event loop phases (in order):**

**1. Timers:**
- Execute `setTimeout`, `setInterval` callbacks
- Runs expired timers

**2. Pending callbacks:**
- I/O callbacks deferred from previous cycle
- System operations

**3. Idle, prepare:**
- Internal use only

**4. Poll:**
- Retrieve new I/O events
- Execute I/O callbacks
- Blocks here if no timers scheduled

**5. Check:**
- `setImmediate()` callbacks

**6. Close callbacks:**
- Close events (socket.on('close'))

**Special queues:**
- **nextTick queue:** Highest priority, before any phase
- **Microtask queue:** Promises, between phases

**Order:** nextTick → Promises → Phase → nextTick → Promises → Next Phase

---

## 3. How would you implement a custom stream in Node.js?

**Extend base classes:**

**Readable stream:**
- Extend `stream.Readable`
- Implement `_read()` method
- Push data or null (end)

**Writable stream:**
- Extend `stream.Writable`
- Implement `_write()` method
- Handle backpressure

**Transform stream:**
- Extend `stream.Transform`
- Implement `_transform()` method
- Optionally `_flush()`

**Considerations:**
- Handle errors
- Manage buffering
- Respect highWaterMark
- Emit appropriate events

---

## 4. What strategies would you use to optimize Node.js application performance?

**Performance strategies:**

**1. Clustering:**
- Use all CPU cores
- PM2 or native cluster module
- Load balance workers

**2. Caching:**
- Redis for shared cache
- In-memory for single server
- HTTP caching headers

**3. Database optimization:**
- Proper indexing
- Connection pooling
- Query optimization
- Read replicas

**4. Async operations:**
- Avoid blocking code
- Use streams for large data
- Worker threads for CPU tasks

**5. Code optimization:**
- Lazy loading
- Efficient algorithms
- Avoid memory leaks
- Minimize dependencies

**6. Infrastructure:**
- CDN for static assets
- Compression (gzip)
- HTTP/2
- Load balancing

---

## 5. How do you handle memory leaks in Node.js applications?

**Common causes:**
- Global variables
- Forgotten timers/intervals
- Event listeners not removed
- Closures holding references
- Large objects in cache

**Detection:**
- Monitor memory usage over time
- Heap snapshots (Chrome DevTools)
- Heap dumps (`--inspect`)
- Production APM tools

**Prevention:**
- Clear timers when done
- Remove event listeners
- Use WeakMap/WeakSet
- Limit cache size
- Profile during development

**Tools:**
- `process.memoryUsage()`
- `--expose-gc` flag
- Clinic.js Doctor
- heapdump package

---

## 6. Explain how to implement a microservices architecture with Node.js

**Key components:**

**1. Service isolation:**
- Each service separate codebase
- Own database (database per service)
- Independent deployment

**2. Communication:**
- REST APIs between services
- Message queues (async)
- gRPC for performance
- Event-driven

**3. Service discovery:**
- Consul, Eureka
- Kubernetes service mesh
- DNS-based

**4. API Gateway:**
- Single entry point
- Routing, authentication
- Rate limiting

**5. Monitoring:**
- Distributed tracing (Jaeger)
- Centralized logging (ELK)
- Health checks

**6. Data consistency:**
- Saga pattern
- Event sourcing
- Eventual consistency

---

## 7. How would you implement a job queue system from scratch?

**Core components:**

**1. Queue storage:**
- Redis lists
- Database table
- Message broker (RabbitMQ)

**2. Job structure:**
- Unique ID
- Type/handler
- Payload data
- Priority
- Retry count
- Status

**3. Worker process:**
- Poll queue
- Execute job
- Update status
- Handle failures

**4. Features needed:**
- Priority queues
- Delayed jobs
- Retry logic
- Dead letter queue
- Concurrency control
- Progress tracking

**Production:** Use Bull, Bee-Queue, or Agenda instead.

---

## 8. What are the trade-offs between different Node.js frameworks (Express, Fastify, NestJS)?

**Express:**
- **Pros:** Mature, huge ecosystem, simple, flexible
- **Cons:** Slower, no built-in TypeScript, less structure
- **Use:** Standard REST APIs, prototypes

**Fastify:**
- **Pros:** Fast, schema validation, modern, plugin system
- **Cons:** Smaller ecosystem, less familiar
- **Use:** Performance-critical APIs

**NestJS:**
- **Pros:** TypeScript-first, structured, Angular-like, great for large teams
- **Cons:** Opinionated, learning curve, heavier
- **Use:** Enterprise apps, microservices, large projects

**Koa:**
- **Pros:** Modern, async/await native, small core
- **Cons:** Less middleware, smaller community
- **Use:** Custom frameworks, experienced teams

---

## 9. How would you implement graceful shutdown in a Node.js application?

**Steps:**

**1. Listen for signals:**
- SIGTERM (Kubernetes)
- SIGINT (Ctrl+C)

**2. Stop accepting new requests:**
- Close HTTP server

**3. Finish pending requests:**
- Give time to complete
- Set timeout

**4. Close connections:**
- Database pools
- Redis clients
- Message queues

**5. Exit cleanly:**
- Exit code 0

**Timeout handling:**
- Force shutdown after grace period
- Exit code 1 if forced

**Kubernetes:** Use preStop hook for grace period.

---

## 10. Explain how to use APM (Application Performance Monitoring) tools effectively

**What to monitor:**

**1. Performance metrics:**
- Response times
- Throughput (requests/sec)
- Error rates
- Apdex score

**2. Resource usage:**
- CPU
- Memory
- Event loop lag
- Garbage collection

**3. Dependencies:**
- Database query times
- External API calls
- Cache hit rates

**4. Business metrics:**
- User actions
- Conversion funnels
- Feature usage

**Popular tools:** New Relic, Datadog, Dynatrace, AppDynamics

**Best practices:**
- Set up alerts
- Create dashboards
- Track trends over time
- Correlate with deployments

---

## 11. How would you implement a real-time notification system at scale?

**Architecture components:**

**1. Connection layer:**
- WebSocket servers
- Socket.io or native WebSocket
- Sticky sessions or Redis adapter

**2. Message broker:**
- Redis Pub/Sub
- RabbitMQ
- Kafka for high volume

**3. User presence:**
- Track connected users
- Redis for distributed state

**4. Delivery:**
- Fan-out to connected clients
- Queue for offline users
- Push notifications fallback

**Scaling considerations:**
- Multiple WebSocket servers
- Shared state in Redis
- Message broker for cross-server
- Connection pooling

---

## 12. What are the considerations for implementing WebSocket connections at scale?

**Challenges:**

**1. Connection management:**
- Each WebSocket is long-lived
- Memory per connection
- File descriptor limits

**2. Load balancing:**
- Sticky sessions required
- IP-based or cookie-based
- Or use Redis adapter

**3. Horizontal scaling:**
- Share state across servers
- Redis Pub/Sub for messages
- Consistent routing

**4. Reconnection:**
- Auto-reconnect logic
- Exponential backoff
- Message queue during disconnect

**5. Security:**
- Authentication
- Rate limiting
- DDoS protection

---

## 13. How would you handle database connection pooling and optimization?

**Connection pooling:**

**Purpose:** Reuse connections instead of creating new ones.

**Configuration:**
- **Pool size:** Balance between throughput and resources
- **Min/max connections**
- **Connection timeout**
- **Idle timeout**

**Optimization:**
- Monitor pool usage
- Adjust size based on load
- Use read replicas
- Connection pooling at app level

**Best practices:**
- Don't close connections manually
- Handle connection errors
- Monitor pool metrics
- Separate pools for different operations

---

## 14. Explain how to implement distributed tracing in a Node.js microservices system

**Distributed tracing:** Track request flow across multiple services.

**Key concepts:**

**1. Trace ID:**
- Unique per request
- Passed between services
- In headers or context

**2. Spans:**
- Represent operations
- Parent-child relationships
- Duration, tags, logs

**3. Context propagation:**
- HTTP headers
- Message metadata
- Automatic instrumentation

**Tools:**
- **Jaeger:** Open source
- **Zipkin:** Twitter's solution
- **OpenTelemetry:** Standard API
- **Datadog, New Relic:** Commercial

**Implementation:**
- Instrument HTTP calls
- Database queries
- External APIs
- Custom operations

---

## 15. How would you design a Node.js application to handle 10,000+ concurrent connections?

**Architecture:**

**1. Clustering:**
- One worker per CPU core
- Shared port via cluster module
- PM2 for management

**2. Load balancer:**
- Nginx or cloud LB
- Distribute across workers
- Health checks

**3. Stateless design:**
- No session in memory
- Redis for session store
- Horizontal scaling ready

**4. Efficient I/O:**
- Use streams
- Avoid blocking operations
- Async everywhere

**5. Resource management:**
- Connection pooling
- Rate limiting
- Backpressure handling

**6. Monitoring:**
- Event loop lag
- Memory usage
- Connection count
- Response times

**Node.js strengths:** Non-blocking I/O perfect for high concurrency with low CPU operations.
