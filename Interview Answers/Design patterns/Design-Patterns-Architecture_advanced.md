# Design Patterns & Architecture - Advanced Level Answers

---

## 1. How would you architect a multi-tenant B2B SaaS application for scalability and data isolation?

**Data isolation strategies:**

**1. Shared database, shared schema:**
- tenant_id in every table/document
- Simplest, most cost-effective
- Risk of data leakage
- Best for: Many small tenants

**2. Shared database, separate schemas:**
- Schema per tenant
- Better isolation
- More complex management
- Best for: Medium-sized tenants

**3. Separate databases:**
- Database per tenant
- Strong isolation
- Easy backup/restore per tenant
- Best for: Enterprise customers

**Hybrid approach:**
- Shared for small tenants
- Dedicated for large/enterprise

**Scalability:**
- Horizontal sharding by tenant
- Read replicas
- Caching layer
- CDN for static assets

---

## 2. Explain Event Sourcing and its trade-offs in a SaaS context

**Event Sourcing:** Store all changes as sequence of events instead of current state.

**How it works:**
- Every state change = event
- Events are immutable, append-only
- Current state = replay events
- Event store is source of truth

**Benefits:**
- Complete audit log
- Time travel (reconstruct past state)
- Event replay
- Multiple projections from same events
- Natural fit for event-driven

**Trade-offs:**
- Complexity (learning curve)
- Storage growth (all events stored)
- Query complexity (need projections)
- Eventual consistency
- Schema evolution challenges

**SaaS context:** Valuable for audit requirements, compliance, analytics.

---

## 3. How would you design a microservices architecture for a suite of related products?

**Design approach:**

**Service boundaries:**
- Domain-driven design
- One service per bounded context
- Shared services (auth, billing)
- Product-specific services

**Communication:**
- Sync: REST/gRPC for queries
- Async: Message queues for commands
- Event bus for notifications

**Data:**
- Database per service
- Shared data via APIs
- Event sourcing for history

**Cross-cutting:**
- API Gateway (routing, auth)
- Service mesh (observability)
- Shared libraries (minimal)

**Challenges:**
- Distributed transactions (Saga pattern)
- Data consistency (eventual)
- Service discovery
- Monitoring complexity

---

## 4. What are the considerations for implementing domain-driven design (DDD)?

**DDD core concepts:**

**1. Ubiquitous Language:**
- Shared vocabulary
- Business and tech alignment

**2. Bounded Contexts:**
- Clear boundaries
- Models per context
- Context mapping

**3. Entities & Value Objects:**
- Identity-based vs value-based
- Encapsulation

**4. Aggregates:**
- Consistency boundaries
- Root entity
- Transaction boundaries

**5. Domain Services:**
- Operations across entities
- Business logic

**6. Repositories:**
- Persistence abstraction

**When to use:**
- Complex domains
- Long-lived projects
- Business-critical logic

**Trade-offs:** Upfront complexity, requires domain expertise.

---

## 5. How would you handle service-to-service communication in a distributed system?

**Synchronous (Request/Response):**
- **REST:** Simple, HTTP-based, widely supported
- **gRPC:** Fast, binary, type-safe, streaming
- **GraphQL:** Flexible queries, single endpoint

**Asynchronous (Messaging):**
- **Message Queue:** Point-to-point, guaranteed delivery
- **Pub/Sub:** One-to-many, event-driven
- **Event Streaming:** Kafka for high throughput

**Patterns:**
- **Request/Reply:** Sync-like over async
- **Saga:** Distributed transactions
- **CQRS:** Separate read/write paths

**Considerations:**
- Latency requirements
- Reliability needs
- Coupling tolerance
- Error handling
- Tracing/monitoring

**Best practice:** Async for commands, sync for queries.

---

## 6. Explain the Saga pattern for distributed transactions

**Saga:** Sequence of local transactions, each publishing event/message to trigger next.

**Two types:**

**Choreography:**
- Decentralized
- Services react to events
- No orchestrator
- More complex to understand

**Orchestration:**
- Central coordinator
- Orchestrator tells services what to do
- Easier to reason about
- Single point of failure

**Compensation:**
- Each step has compensating transaction
- Rollback via compensation

**Use case:** Order placement across payment, inventory, shipping services.

**Challenges:** Eventual consistency, complex debugging, compensation logic.

---

## 7. How would you implement a plugin architecture for extensibility?

**Design:**

**1. Plugin interface:**
- Define hooks/extension points
- Standard API contract
- Versioning

**2. Discovery:**
- Plugin registry
- Auto-discovery (scan directory)
- Manifest files

**3. Lifecycle:**
- Install/uninstall
- Enable/disable
- Dependencies

**4. Isolation:**
- Sandboxing
- Resource limits
- Error boundaries

**5. Communication:**
- Event system
- Shared services
- Context passing

**Examples:** VS Code extensions, WordPress plugins, Webpack loaders.

**Security:** Validate plugins, permissions, code review.

---

## 8. What strategies would you use for API versioning in a fast-growing product?

**Versioning strategies:**

**1. URL versioning:**
- /v1/users, /v2/users
- Clear, explicit
- Can duplicate code

**2. Header versioning:**
- Accept: application/vnd.api+json;version=2
- URLs stay clean
- Less visible

**3. Query parameter:**
- /users?version=2
- Simple
- Easy to miss

**Best practices:**
- Support multiple versions temporarily
- Deprecation warnings
- Migration guide
- Version sunset timeline
- Monitoring version usage

**SaaS context:** Gradual rollout, per-tenant versioning, feature flags.

---

## 9. How would you design a system for zero-downtime deployments?

**Strategies:**

**1. Blue-Green Deployment:**
- Two identical environments
- Switch traffic atomically
- Easy rollback

**2. Rolling Deployment:**
- Update instances gradually
- Always some instances available
- Lower resource needs

**3. Canary Deployment:**
- Small subset gets new version
- Monitor before full rollout
- Early problem detection

**Infrastructure:**
- Load balancer for traffic switching
- Health checks
- Graceful shutdown
- Database migrations (backward compatible)

**State management:**
- Stateless services
- External session storage
- Database migrations run first

**Monitoring:** Track errors, performance during deployment.

---

## 10. Explain the CQRS + Event Sourcing pattern and its implementation challenges

**CQRS + Event Sourcing combined:**
- Commands → Events → Event Store
- Events project to read models
- Complete audit trail

**Benefits:**
- Optimized read/write models
- Full history
- Scalability
- Multiple projections

**Challenges:**

**1. Eventual consistency:**
- Read model lag
- UI implications

**2. Event schema evolution:**
- Versioning events
- Upcasting old events

**3. Complexity:**
- Learning curve
- More moving parts

**4. Debugging:**
- Harder to trace
- Event replay needed

**5. Projections:**
- Rebuilding can be slow
- Multiple to maintain

**When justified:** Complex domains, audit requirements, high scale.

---

## 11. How would you implement eventual consistency in a distributed system?

**Approaches:**

**1. Event-driven:**
- Publish events on changes
- Services subscribe and update
- At-least-once delivery

**2. Saga pattern:**
- Coordinate multi-service transactions
- Compensation for failures

**3. Read-your-own-writes:**
- User sees their changes immediately
- Even if not globally consistent

**4. Conflict resolution:**
- Last-write-wins
- Custom merge logic
- CRDTs (Conflict-free Replicated Data Types)

**Handling in UI:**
- Optimistic updates
- Loading states
- Retry logic
- Show stale data indicators

**Trade-off:** Complexity for availability and partition tolerance (CAP theorem).

---

## 12. What are the trade-offs between monolith and microservices for a 130-person company?

**Monolith:**
- **Pros:** Simple deployment, easier development, consistent, less overhead
- **Cons:** Scaling limits, tight coupling, deployment risk
- **Good for:** Early-stage, simple domains, small teams

**Microservices:**
- **Pros:** Independent scaling, technology flexibility, team autonomy, resilience
- **Cons:** Complexity, distributed system challenges, operational overhead
- **Good for:** Large scale, multiple teams, complex domains

**For 130-person company:**
- Modular monolith first
- Extract services as needed
- Hybrid approach (monolith + key services)
- Consider team structure (Conway's Law)

**Recommendation:** Start monolith, evolve to microservices when clear boundaries emerge.

---

## 13. How would you design a system to handle multiple products with shared infrastructure?

**Shared components:**
- Authentication/authorization
- Billing/subscription
- User management
- Notification service
- Analytics

**Product isolation:**
- Separate services per product
- Shared libraries (carefully)
- Feature flags per product

**Data strategy:**
- Shared user database
- Product-specific databases
- Cross-product analytics warehouse

**Deployment:**
- Monorepo or multi-repo
- Shared CI/CD pipelines
- Independent deployments

**Challenges:**
- Version compatibility
- Shared service changes
- Cross-product features

**Benefits:** Reduced duplication, consistent UX, operational efficiency.

---

## 14. Explain how you would implement a circuit breaker pattern

**Circuit Breaker:** Prevents cascading failures by stopping calls to failing services.

**States:**
1. **Closed:** Normal operation, requests flow
2. **Open:** Failing, reject requests immediately
3. **Half-Open:** Test if service recovered

**Transitions:**
- Closed → Open: After threshold failures
- Open → Half-Open: After timeout
- Half-Open → Closed: Success
- Half-Open → Open: Still failing

**Implementation:**
- Track failure rate
- Timeout for open state
- Test requests in half-open
- Fallback behavior

**Benefits:** Fast failure, prevent resource exhaustion, give failing service time to recover.

---

## 15. How would you architect a system for multi-region deployment and data residency?

**Architecture:**

**1. Regional isolation:**
- Deploy full stack per region
- Data stays in region
- Compliance with GDPR, etc.

**2. Global services:**
- CDN for static assets
- Global load balancer (DNS/Anycast)
- Central authentication (or regional)

**3. Data synchronization:**
- What needs to be global vs regional
- Async replication
- Conflict resolution

**4. Routing:**
- Geographic routing
- User affinity (user → region)
- Latency-based routing

**Challenges:**
- Data consistency across regions
- Cross-region features
- Cost (duplicate infrastructure)
- Operational complexity

**Data residency:** Store user data in specified region, metadata can be global.
