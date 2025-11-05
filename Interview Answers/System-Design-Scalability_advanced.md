# System Design & Scalability - Advanced Level Answers

---

## 1. How would you design a system to handle $40M ARR scale with 130 employees?

**Architecture principles:**

**1. Proven technology:**
- Mature stacks (Node.js, React, PostgreSQL/MongoDB)
- Avoid bleeding edge
- Team expertise matters

**2. Modular monolith first:**
- Start simple
- Extract services as needed
- Clear module boundaries

**3. Horizontal scalability:**
- Stateless services
- Load balancing
- Auto-scaling

**4. Multi-tenancy:**
- Shared infrastructure
- Tenant isolation
- Per-tenant customization

**5. Observability:**
- Comprehensive monitoring
- Distributed tracing
- Alert fatigue prevention

**Team efficiency:**
- CI/CD automation
- Self-service tools
- Clear ownership
- On-call rotation

**Cost optimization:**
- Cloud-native
- Right-size resources
- Reserved instances
- Caching aggressively

---

## 2. Explain how you would architect a globally distributed application

**Architecture:**

**1. Multi-region deployment:**
- Deploy in multiple geographic regions
- Closer to users = lower latency

**2. Global load balancing:**
- DNS-based (Route53, Cloudflare)
- Anycast routing
- Latency-based or geo-based

**3. Data strategy:**

**Active-Active:**
- Read/write in all regions
- Conflict resolution needed
- Best for global scale

**Active-Passive:**
- Write to primary region
- Replicate to others
- Simpler consistency

**Regional data:**
- Data residency per region
- Comply with regulations

**4. CDN:**
- Static assets globally cached
- Edge computing

**5. Database:**
- Multi-region replication
- Regional read replicas
- CockroachDB, DynamoDB Global Tables

**Challenges:** Consistency, cross-region latency, costs, complexity.

---

## 3. How would you design a system for 99.99% uptime?

**99.99% uptime = 52 minutes downtime/year**

**Strategies:**

**1. Redundancy:**
- No single points of failure
- Multiple availability zones
- Load balanced instances

**2. Health checks:**
- Liveness and readiness
- Automatic failover
- Remove unhealthy instances

**3. Graceful degradation:**
- Core features always work
- Non-essential features can fail

**4. Database:**
- Replication (primary-replica)
- Automatic failover
- Backups and recovery

**5. Deployment:**
- Zero-downtime deploys
- Canary or blue-green
- Automated rollback

**6. Monitoring:**
- Real-time alerts
- SLA tracking
- Incident response

**7. Testing:**
- Chaos engineering
- Load testing
- Failure scenarios

**8. Dependencies:**
- Circuit breakers
- Timeouts
- Fallbacks

---

## 4. What strategies would you use for database scaling (read replicas, sharding, partitioning)?

**Read replicas:**
- Distribute read traffic
- Async replication from primary
- Good for read-heavy workloads
- Eventual consistency for reads

**Sharding (horizontal partitioning):**
- Split data across multiple databases
- Shard key determines distribution
- Challenges: Cross-shard queries, rebalancing
- Use when: Single database can't handle load

**Vertical partitioning:**
- Split tables by columns
- Separate frequently vs rarely accessed
- Reduce table size

**Connection pooling:**
- Reuse connections
- Limit concurrent connections
- Better resource usage

**Caching:**
- Redis for hot data
- Reduce database load

**Indexes:**
- Speed up queries
- Trade-off with write performance

**Choosing strategy:** Read replicas for reads, sharding for data size, caching for hot data.

---

## 5. How would you implement a real-time analytics system?

**Architecture:**

**1. Data collection:**
- Event streaming (Kafka)
- Track user actions
- High throughput

**2. Stream processing:**
- Apache Flink, Spark Streaming
- Real-time aggregations
- Windowing (5-minute, hourly)

**3. Storage:**
- Time-series database (InfluxDB, TimescaleDB)
- Data warehouse (Snowflake, BigQuery) for history
- Hot storage vs cold storage

**4. Querying:**
- Pre-computed metrics
- Materialized views
- Low-latency queries

**5. Visualization:**
- Real-time dashboards
- WebSocket updates
- Grafana, custom dashboards

**Scaling:**
- Partitioned streams
- Distributed processing
- Sampling for high-volume

**Use case:** User behavior analytics, monitoring, business metrics.

---

## 6. Explain how you would design a system to handle millions of webhook deliveries

**Architecture:**

**1. Ingestion:**
- Queue all webhooks (SQS, Kafka)
- Don't block source operations

**2. Processing:**
- Worker pool consuming queue
- Parallel processing
- Horizontal scaling

**3. Delivery:**
- HTTP client with timeouts
- Retry logic (exponential backoff)
- Circuit breaker for failing endpoints

**4. Ordering:**
- FIFO queues if order matters
- Per-destination ordering

**5. Monitoring:**
- Track delivery success rate
- Failed webhook handling
- Latency metrics

**6. Security:**
- HMAC signatures
- Rate limit per endpoint

**7. Reliability:**
- Dead letter queue
- Manual retry interface
- Delivery guarantees (at-least-once)

**Scaling:** Partition by destination, more workers, queue sharding.

---

## 7. How would you architect a system for data encryption at rest and in transit?

**At Rest:**

**1. Database:**
- Transparent Data Encryption (TDE)
- Column-level encryption for sensitive
- Encrypted backups

**2. File storage:**
- Encrypt before storing (S3, disk)
- Server-side or client-side encryption
- Key management (KMS, Vault)

**3. Backups:**
- Encrypted snapshots
- Separate encryption keys

**In Transit:**

**1. TLS/SSL:**
- HTTPS for all APIs
- Minimum TLS 1.2
- Strong ciphers

**2. Service-to-service:**
- mTLS (mutual TLS)
- API gateways
- Service mesh (Istio)

**3. Database connections:**
- Encrypted connections
- SSL certificates

**Key Management:**
- Rotate keys regularly
- Separate keys per tenant (multi-tenant)
- Hardware Security Modules (HSM)
- Key hierarchy

**Compliance:** GDPR, HIPAA, PCI-DSS requirements.

---

## 8. What are the considerations for implementing GDPR compliance in a global SaaS?

**Key requirements:**

**1. Data minimization:**
- Collect only necessary data
- Clear purpose
- Regular audits

**2. Right to access:**
- Export user data
- Machine-readable format
- API for requests

**3. Right to erasure:**
- Delete all user data
- Hard delete, not soft
- Cascade to backups

**4. Data portability:**
- Export in standard format
- Transfer to other services

**5. Consent:**
- Explicit opt-in
- Granular choices
- Easy to withdraw

**6. Data protection:**
- Encryption
- Access controls
- Audit logs

**7. Data residency:**
- Store in user's region
- Cross-border transfers (Privacy Shield)

**8. Breach notification:**
- 72-hour reporting
- Incident response plan

**9. DPO (Data Protection Officer):**
- Required for certain companies
- Privacy oversight

**Technical:**
- Data anonymization
- Audit trails
- Automated deletion
- Privacy by design

---

## 9. How would you design a disaster recovery and backup strategy?

**Backup strategy:**

**1. 3-2-1 Rule:**
- 3 copies of data
- 2 different media types
- 1 offsite copy

**2. Frequency:**
- Continuous (databases)
- Daily (full backups)
- Hourly (incremental)

**3. Retention:**
- Daily for 7 days
- Weekly for 4 weeks
- Monthly for 12 months
- Yearly for compliance

**4. Testing:**
- Regular restore tests
- Verify integrity
- Practice recovery

**Disaster Recovery:**

**1. RTO (Recovery Time Objective):**
- How long to restore
- Different tiers for different services

**2. RPO (Recovery Point Objective):**
- How much data loss acceptable
- Determines backup frequency

**3. Multi-region:**
- Replicate to separate region
- Failover capability
- DNS switching

**4. Runbooks:**
- Documented procedures
- Automated where possible
- Regular drills

**5. Monitoring:**
- Backup success/failure
- Backup age
- Storage usage

---

## 10. Explain how you would implement a feature rollout system with gradual deployment

**Gradual rollout approach:**

**1. Feature flags:**
- Toggle features without deployment
- Per-user, per-tenant, percentage

**2. Canary deployment:**
- Small percentage of users first
- Monitor metrics
- Gradual increase

**3. A/B testing:**
- Compare versions
- Measure impact
- Data-driven decisions

**4. Ring deployment:**
- Internal → Beta → Production
- Increasing circles

**Implementation:**

**1. Feature flag service:**
- Centralized (LaunchDarkly, Unleash)
- Or database-backed
- Real-time updates

**2. Targeting:**
- User attributes
- Tenant segments
- Random percentage

**3. Monitoring:**
- Track feature performance
- Error rates
- User feedback

**4. Rollback:**
- Instant disable
- No redeployment needed

**Best practice:** Start 1%, monitor, 10%, 50%, 100%.

---

## 11. How would you handle data consistency across multiple products in the suite?

**Approaches:**

**1. Shared services:**
- User service
- Billing service
- Shared across products

**2. Event-driven:**
- Publish events on changes
- Products subscribe
- Eventual consistency

**3. Shared database:**
- Common tables (users, tenants)
- Product-specific tables separate

**4. API contracts:**
- Products communicate via APIs
- Versioned interfaces
- Backward compatibility

**5. Data synchronization:**
- CDC (Change Data Capture)
- Stream changes (Debezium)
- Real-time or batch sync

**Challenges:**
- Different update frequencies
- Schema evolution
- Circular dependencies

**Trade-offs:** Strong consistency vs autonomy, coupling vs duplication.

---

## 12. What strategies would you use for optimizing database costs at scale?

**Cost reduction strategies:**

**1. Right-sizing:**
- Monitor usage
- Scale down oversized instances
- Reserved instances for savings

**2. Storage optimization:**
- Archive old data
- Compress data
- Delete soft-deleted records

**3. Query optimization:**
- Indexes for slow queries
- Reduce full table scans
- Limit result sizes

**4. Caching:**
- Redis for hot data
- Reduce database load
- Longer TTLs where acceptable

**5. Read replicas:**
- Cheaper instances for reads
- Offload from primary

**6. Partitioning:**
- Separate hot and cold data
- Cheaper storage for archives

**7. Connection pooling:**
- Reduce connections
- Lower instance requirements

**8. Monitoring:**
- Track costs per query
- Identify expensive operations
- Optimize high-cost queries

**Cloud-specific:** Auto-scaling, serverless (Aurora Serverless), spot instances.

---

## 13. How would you design a system to support multiple API versions simultaneously?

**Versioning strategies:**

**1. URL-based:**
- /v1/users, /v2/users
- Clear and explicit
- Easy routing

**2. Header-based:**
- Accept: application/vnd.api.v2+json
- URLs stay clean

**Implementation:**

**1. Shared code:**
- Extract common logic
- Version-specific adapters
- Transformers for differences

**2. Separate routers:**
- Route by version
- Different controllers
- Shared services

**3. Database schema:**
- Support all versions
- Migrations backward compatible
- Version-specific views

**4. Testing:**
- Test all supported versions
- Integration tests per version

**Lifecycle:**
- Support N versions
- Deprecation warnings
- Sunset timeline (6-12 months)
- Monitor version usage

**Documentation:** Separate docs per version, migration guides.

---

## 14. Explain how you would implement observability (logs, metrics, traces) at scale

**Three pillars:**

**1. Logs:**
- Structured (JSON)
- Centralized (ELK, Splunk)
- Sampling for high-volume
- Include trace IDs

**2. Metrics:**
- Time-series data
- Prometheus, Datadog
- Business and technical metrics
- Pre-aggregated

**3. Traces:**
- Distributed tracing
- Jaeger, Zipkin
- Request flow across services
- Performance bottlenecks

**Implementation:**

**1. Instrumentation:**
- Auto-instrumentation (OpenTelemetry)
- Custom metrics
- Consistent naming

**2. Correlation:**
- Trace ID across services
- Link logs to traces
- Request context propagation

**3. Sampling:**
- 100% errors
- Sample successes
- Adaptive sampling

**4. Dashboards:**
- Service health
- SLIs/SLOs
- Business metrics

**5. Alerting:**
- Threshold-based
- Anomaly detection
- Alert on SLO violations

**At scale:** Sampling, efficient storage, retention policies, cost management.

---

## 15. How would you architect a system to support white-labeling for enterprise customers?

**White-labeling components:**

**1. Branding:**
- Custom domain (CNAME)
- Logo, colors, fonts
- Stored per tenant
- CSS variables or themes

**2. Email templates:**
- Branded email templates
- Customer logo
- Custom copy

**3. Multi-tenancy:**
- Tenant configuration
- Feature flags per tenant
- Custom workflows

**4. Infrastructure:**
- Subdomain per customer (tenant.yourapp.com)
- Custom domain (app.customer.com)
- SSL certificates (wildcard or per-domain)

**5. API:**
- Tenant-scoped API keys
- Custom webhooks
- Branded API docs

**6. Data isolation:**
- Separate databases (enterprise tier)
- Or tenant_id with strict filtering

**Implementation:**

**1. Configuration:**
- Tenant settings table
- Asset storage (S3)
- CDN for assets

**2. Rendering:**
- Dynamic themes
- Server-side rendering with tenant context
- Client-side theme loading

**3. Deployment:**
- Shared infrastructure
- Custom code per tenant (plugins)

**Pricing:** Typically higher tier, custom contracts.
