# System Design & Scalability - Intermediate Level Answers

---

## 1. How would you implement a caching strategy for a SaaS application?

**Multi-layer caching:**

**1. Browser cache:**
- Static assets
- Cache-Control headers
- Versioned URLs

**2. CDN:**
- Images, CSS, JS
- Edge caching

**3. Application cache (Redis):**
- Database query results
- Session data
- Computed values

**4. Database cache:**
- Query result cache

**Strategy:**
- Cache-aside (lazy loading)
- Write-through (update cache on write)
- TTL (Time To Live)

**Invalidation:**
- Time-based expiration
- Event-based (on updates)
- Cache keys with versions

---

## 2. Explain different types of caching (in-memory, distributed, HTTP)

**In-memory:**
- Store in application RAM
- Fastest access
- Lost on restart
- Single server only
- Example: JavaScript Map, variables

**Distributed (Redis, Memcached):**
- Shared across servers
- Persistent or in-memory
- Networked access (slower than in-memory)
- Scalable
- Survives app restarts

**HTTP caching:**
- Browser and proxy caching
- Headers: Cache-Control, ETag
- CDN caching
- Reduces server requests

**When to use each:**
- In-memory: Frequently accessed, small data
- Distributed: Shared data, sessions
- HTTP: Static assets, public content

---

## 3. How would you design a rate limiting system?

**Algorithms:**

**1. Fixed window:**
- Count requests per time window
- Simple but traffic spikes at edges

**2. Sliding window:**
- Smooth rate over time
- More accurate
- More complex

**3. Token bucket:**
- Tokens refill at fixed rate
- Burst capacity
- Smooth long-term rate

**4. Leaky bucket:**
- Process at fixed rate
- Queue excess requests

**Implementation:**
- Store counters in Redis
- Atomic increment operations
- Per-user, per-IP, or per-API key
- Multiple limits (per-minute, per-hour)

**Response:** Return remaining quota in headers.

---

## 4. What are the considerations for implementing a job queue?

**Requirements:**

**1. Reliability:**
- Persistent storage
- At-least-once delivery
- Retry failed jobs

**2. Priority:**
- Multiple queues by priority
- High priority processed first

**3. Concurrency:**
- Worker pool
- Parallel processing
- Limit concurrent jobs

**4. Monitoring:**
- Queue length
- Processing time
- Failed jobs
- Worker health

**Features:**
- Delayed jobs
- Scheduled jobs
- Job dependencies
- Dead letter queue

**Tools:** Bull (Redis), RabbitMQ, AWS SQS.

---

## 5. How would you implement real-time notifications at scale?

**Architecture:**

**1. WebSocket servers:**
- Persistent connections
- Multiple servers for scale
- Sticky sessions or Redis adapter

**2. Message broker:**
- Redis Pub/Sub
- RabbitMQ
- Fan-out to connected clients

**3. Fallback:**
- Push notifications (mobile)
- Email (offline users)
- Poll-based (graceful degradation)

**Scaling:**
- Multiple WebSocket servers
- Load balancer with sticky sessions
- Share state via Redis
- Message broker for cross-server

**Optimization:**
- Only send relevant notifications
- Batch messages
- Compress large payloads

---

## 6. Explain the CAP theorem and its implications

**CAP Theorem:** Distributed system can only guarantee 2 of 3:

**Consistency:**
- All nodes see same data
- Read after write returns new value

**Availability:**
- Every request gets response
- No downtime

**Partition Tolerance:**
- System continues despite network partitions
- Network can fail

**Implication:** Must choose during partition:
- **CP:** Consistency + Partition (sacrifice availability)
- **AP:** Availability + Partition (sacrifice consistency)
- **CA:** Not realistic (networks partition)

**Real-world:**
- Traditional databases: CP
- NoSQL (Cassandra, DynamoDB): AP
- Many systems: Tunable consistency

---

## 7. How would you design a logging and monitoring system?

**Logging:**

**1. Structured logging:**
- JSON format
- Include context (request ID, user ID)
- Appropriate levels

**2. Centralization:**
- Log aggregation (ELK, Splunk)
- Easy searching
- Correlation

**3. Retention:**
- Storage limits
- Archive old logs
- Compliance requirements

**Monitoring:**

**1. Metrics:**
- Response times
- Error rates
- Resource usage

**2. Alerts:**
- Threshold-based
- Anomaly detection
- On-call rotations

**3. Dashboards:**
- Real-time visibility
- Historical trends

**4. Tracing:**
- Distributed tracing
- Request flows

**Tools:** Prometheus, Grafana, Datadog, New Relic.

---

## 8. What strategies would you use for handling peak traffic?

**Preparation:**

**1. Horizontal scaling:**
- Auto-scaling
- Pre-provision for known peaks

**2. Caching:**
- Aggressive caching
- Warm caches before peak

**3. CDN:**
- Static assets on CDN
- Reduce origin load

**4. Database:**
- Read replicas
- Connection pooling
- Query optimization

**During peak:**

**1. Rate limiting:**
- Protect system
- Prioritize critical operations

**2. Queue:**
- Buffer non-critical requests
- Process async

**3. Degrade gracefully:**
- Disable non-essential features
- Show cached data

**4. Load shedding:**
- Reject excess traffic
- Better than crashing

---

## 9. How would you implement search functionality in a large dataset?

**Approaches:**

**1. Database full-text search:**
- PostgreSQL full-text
- MongoDB text indexes
- Good for simple searches

**2. Elasticsearch:**
- Dedicated search engine
- Advanced features
- Scales well
- Sync from primary database

**3. Indexing strategy:**
- Index frequently searched fields
- Optimize queries
- Denormalize for search

**4. Features:**
- Autocomplete
- Fuzzy matching
- Faceted search
- Relevance scoring

**Optimization:**
- Pagination
- Limit results
- Caching popular searches
- Background indexing

---

## 10. Explain the difference between optimistic and pessimistic locking

**Optimistic locking:**
- Assume conflicts rare
- Check version before update
- Fail if version changed
- Better concurrency

**Pessimistic locking:**
- Lock resource for entire transaction
- Block concurrent access
- Guaranteed no conflicts
- Can cause contention

**When to use:**
- **Optimistic:** Low contention, read-heavy
- **Pessimistic:** High contention, critical updates

**Implementation:**
- Optimistic: Version number/timestamp
- Pessimistic: Database locks (SELECT FOR UPDATE)

---

## 11. How would you design a system for handling file uploads and storage?

**Upload handling:**

**1. Size limits:**
- Prevent abuse
- Memory concerns

**2. Validation:**
- File type (MIME)
- Virus scanning
- Content validation

**3. Upload methods:**
- Direct upload (small files)
- Multipart (large files)
- Resumable uploads
- Pre-signed URLs (S3)

**Storage:**

**1. Local:**
- Simple for small scale
- Hard to scale

**2. Object storage (S3):**
- Scalable
- Durable
- CDN integration

**Processing:**
- Async (queues)
- Thumbnails, conversions
- Metadata extraction

**Security:**
- Random filenames
- Access control
- HTTPS

---

## 12. What are the considerations for implementing webhooks?

**Sending webhooks:**

**1. Reliability:**
- Retry logic (exponential backoff)
- Timeout handling
- Dead letter queue for failures

**2. Security:**
- HMAC signatures
- Verify receiver
- HTTPS only

**3. Delivery:**
- Async processing
- Queue for scaling
- Ordered delivery (optional)

**Receiving webhooks:**

**1. Verification:**
- Validate signatures
- Check timestamp (prevent replay)

**2. Idempotency:**
- Handle duplicates
- Use event IDs

**3. Processing:**
- Quick acknowledgment (200 OK)
- Process async
- Handle errors gracefully

---

## 13. How would you implement a multi-tenant database strategy?

**Strategies:**

**1. Shared database, shared schema:**
- tenant_id column
- Filter all queries
- Pros: Simple, cost-effective
- Cons: Risk of leakage

**2. Shared database, separate schemas:**
- Schema per tenant
- Better isolation
- Pros: Moderate isolation
- Cons: Management complexity

**3. Separate databases:**
- Database per tenant
- Pros: Strong isolation, easy backup
- Cons: Higher cost, management

**Best practice:**
- Start with shared schema
- Migrate large tenants to dedicated

**Security:**
- Row-level security
- Strict query filtering
- Audit all data access

---

## 14. Explain the benefits and drawbacks of serverless architecture

**Benefits:**

**1. Cost:**
- Pay per execution
- No idle server costs

**2. Scaling:**
- Auto-scaling
- No capacity planning

**3. Management:**
- No server maintenance
- Focus on code

**Drawbacks:**

**1. Cold starts:**
- Initial latency
- Inconsistent performance

**2. Limits:**
- Execution time limits
- Memory constraints
- Concurrent execution caps

**3. Vendor lock-in:**
- Platform-specific
- Hard to migrate

**4. Complexity:**
- Distributed debugging
- State management

**Best for:** Event-driven, variable load, background jobs
**Not for:** Long-running, consistent latency needs

---

## 15. How would you design an API for third-party integrations?

**Design principles:**

**1. Versioning:**
- URL or header versioning
- Backwards compatibility
- Deprecation policy

**2. Authentication:**
- API keys
- OAuth for user data
- Rate limits per key

**3. Documentation:**
- Clear, comprehensive
- Examples
- SDKs in popular languages
- Interactive playground

**4. Rate limiting:**
- Per-client limits
- Different tiers

**5. Error handling:**
- Consistent error format
- Meaningful messages
- Error codes

**6. Webhooks:**
- Event notifications
- Configurable endpoints

**7. Testing:**
- Sandbox environment
- Test credentials
- Mock data

**Monitoring:** Track usage, errors, adoption per endpoint.
