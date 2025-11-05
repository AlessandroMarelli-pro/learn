# Testing & Quality - All Levels

---

## Beginner Level

### 1. What is the difference between unit tests, integration tests, and e2e tests?

**Unit tests:**
- Test individual functions/methods in isolation
- Fast, many tests
- Mock dependencies
- Test logic, edge cases

**Integration tests:**
- Test multiple components together
- Database, API interactions
- Medium speed
- Real dependencies or test doubles

**End-to-end (E2E) tests:**
- Test entire application flow
- User perspective
- Slow, fewer tests
- Browser automation
- Real environment

**Pyramid:** Many unit, some integration, few E2E.

---

### 2. What is Test-Driven Development (TDD)?

**TDD:** Write tests before code.

**Red-Green-Refactor cycle:**
1. **Red:** Write failing test
2. **Green:** Write minimal code to pass
3. **Refactor:** Improve code quality

**Benefits:**
- Forces testable code
- Living documentation
- Confidence in changes
- Fewer bugs

**Challenges:** Learning curve, slower initially, not always practical.

---

### 3. What testing frameworks have you used in JavaScript?

**Common frameworks:**

**Test runners:**
- **Jest:** Popular, all-in-one, zero config
- **Mocha:** Flexible, needs assertion library
- **Vitest:** Fast, Vite-based

**Assertion libraries:**
- **Chai:** Expect/should syntax
- **Jest:** Built-in assertions

**Mocking:**
- **Sinon:** Spies, stubs, mocks
- **Jest:** Built-in mocking

**E2E:**
- **Cypress:** Modern, developer-friendly
- **Playwright:** Cross-browser
- **Puppeteer:** Headless Chrome

---

### 4. How do you mock dependencies in tests?

**Mocking:** Replace real dependencies with test doubles.

**Types:**

**Stub:** Returns predetermined values
**Spy:** Records how it was called
**Mock:** Pre-programmed expectations

**Jest example:**
- `jest.fn()`: Create mock function
- `jest.mock()`: Mock entire module
- `jest.spyOn()`: Spy on method

**Why mock:**
- Isolate unit under test
- Control behavior
- Avoid external dependencies
- Speed up tests

---

### 5. What is code coverage and why does it matter?

**Code coverage:** Percentage of code executed by tests.

**Metrics:**
- **Line coverage:** Lines executed
- **Branch coverage:** If/else paths
- **Function coverage:** Functions called
- **Statement coverage:** Statements executed

**Why it matters:**
- Identify untested code
- Measure test completeness
- Quality indicator

**Warning:** 100% coverage ≠ bug-free, quality > quantity.

**Tools:** Istanbul, Jest --coverage, nyc.

---

### 6-15. Quick Beginner Answers

**6. Assertions:** Statements verifying expected outcome (expect, assert)

**7. Test fixtures:** Setup data/state before tests

**8. Testing async code:** async/await in tests, done callback, return promises

**9. Stub vs spy:** Stub replaces behavior, spy watches real function

**10. Continuous Integration (CI):** Automated testing on every commit

**11. Testing REST APIs:** Supertest, request libraries, validate responses

**12. Linting:** Static code analysis (ESLint), catch errors, enforce style

**13. Testing pyramid:** Many unit → some integration → few E2E

**14. Test data:** Factories, fixtures, random generation, separate from production

**15. Snapshot tests:** Save component output, detect unintended changes

---

## Intermediate Level

### 1. How would you structure tests in a Node.js application?

**Folder structure:**
```
src/
  users/
    user.service.js
    user.service.test.js
    user.controller.js
    user.controller.test.js
tests/
  integration/
    user.api.test.js
  e2e/
    user-flow.test.js
```

**Organization:**
- Tests next to source OR
- Separate test directory
- Mirror source structure

**Naming:**
- `*.test.js` or `*.spec.js`
- Descriptive names

**Setup:**
- Global setup/teardown
- Test helpers
- Shared fixtures

---

### 2. Explain the difference between mocks, stubs, and spies

**Stub:**
- Provides canned responses
- Replaces real implementation
- No verification
- Controls behavior

**Spy:**
- Wraps real function
- Records calls
- Passes through or stubs
- Verifies how called

**Mock:**
- Stub + expectations
- Pre-programmed behavior and verification
- Fails if expectations not met

**When to use:**
- **Stub:** Control dependencies
- **Spy:** Verify calls
- **Mock:** Complex expectations

---

### 3. How do you test database interactions?

**Approaches:**

**1. Test database:**
- Separate database for tests
- Reset between tests
- Transactions (rollback after test)

**2. In-memory database:**
- SQLite in-memory
- MongoDB Memory Server
- Fast, isolated

**3. Mocking:**
- Mock database calls
- Test logic only
- Fast but not comprehensive

**4. Containers:**
- Docker for test database
- Consistent environment

**Best practice:** Integration tests use real DB, unit tests mock.

---

### 4. What strategies do you use for testing error scenarios?

**Testing errors:**

**1. Expected errors:**
- Validate error thrown
- Check error message/type
- Test error handling code

**2. Unexpected errors:**
- Network failures
- Database down
- Timeout scenarios

**Techniques:**
- Mock dependencies to throw
- Force error conditions
- Test retry logic
- Verify logging
- Check user-facing messages

**Coverage:** Test happy path AND error paths.

---

### 5. How would you implement contract testing for APIs?

**Contract testing:** Verify API provider and consumer agree on interface.

**Approaches:**

**1. Pact:**
- Consumer defines expectations
- Provider verifies can meet them
- Stored in "Pact Broker"

**2. OpenAPI/Swagger:**
- Define API spec
- Generate tests from spec
- Validate requests/responses

**Benefits:**
- Catch breaking changes early
- Independent service deployment
- Living documentation

**Process:**
1. Consumer writes contract tests
2. Provider runs contract tests
3. Both must pass for compatibility

---

### 6-15. Intermediate Quick Answers

**6. Testing middleware:** Mock req/res/next, verify behavior

**7. Testing auth/authorization:** Mock user context, test permissions

**8. Mutation testing:** Change code, verify tests fail (test quality)

**9. Testing WebSockets:** Mock connections, test event handlers

**10. Testing event-driven code:** Verify events emitted, handlers called

**11. Handling flaky tests:** Isolate, fix timing issues, avoid randomness

**12. Integration testing in microservices:** Test service boundaries, contracts

**13. Testing file uploads:** Mock multer, temp files, verify processing

**14. Property-based testing:** Generate random inputs, verify properties hold

**15. E2E tests for SaaS:** Test critical flows, user journeys, multi-tenant

---

## Advanced Level

### 1. How would you design a testing strategy for a suite of B2B SaaS products?

**Strategy:**

**1. Test pyramid:**
- 70% unit tests (fast, many)
- 20% integration tests (API, database)
- 10% E2E tests (critical flows only)

**2. Per-product:**
- Product-specific tests
- Shared library tests
- Contract tests between products

**3. Multi-tenancy:**
- Test tenant isolation
- Test cross-tenant scenarios don't work
- Different tenant configurations

**4. Automation:**
- CI/CD pipeline
- Run on every PR
- Parallel execution

**5. Performance:**
- Load testing critical endpoints
- Database query performance
- Frontend performance budgets

**6. Security:**
- OWASP testing
- Penetration testing
- Dependency scanning

---

### 2. Explain how you would implement chaos engineering practices

**Chaos engineering:** Deliberately inject failures to test resilience.

**Practices:**

**1. Gamedays:**
- Scheduled failure injection
- Team observes and responds
- Learning exercises

**2. Failure injection:**
- Kill random services
- Network latency/partitions
- Database unavailability
- Resource exhaustion

**3. Tools:**
- Chaos Monkey (Netflix)
- Gremlin
- Chaos Toolkit

**4. Start small:**
- Non-production first
- Controlled experiments
- Hypothesis-driven

**5. Monitor:**
- Observe system behavior
- Measure blast radius
- Verify graceful degradation

**Goal:** Build confidence in system resilience.

---

### 3. How would you test multi-tenant isolation?

**Isolation tests:**

**1. Data access:**
- Verify Tenant A can't read Tenant B data
- Test all API endpoints
- Try direct database queries

**2. Authentication:**
- Token from Tenant A shouldn't work for Tenant B
- Cross-tenant user access

**3. Rate limiting:**
- One tenant can't exhaust another's quota

**4. Resources:**
- CPU/memory isolation
- Database connection pools

**Automated tests:**
- Create test tenants
- Attempt cross-tenant operations
- Verify 403/404, not data

**Manual:** Penetration testing, security audit.

---

### 4. What strategies would you use for performance testing at scale?

**Types:**

**1. Load testing:**
- Expected traffic levels
- Verify performance targets
- Tools: k6, Artillery, JMeter

**2. Stress testing:**
- Beyond capacity
- Find breaking point

**3. Spike testing:**
- Sudden traffic increase
- Test auto-scaling

**4. Soak testing:**
- Sustained load over time
- Memory leaks, degradation

**Metrics:**
- Response times (p50, p95, p99)
- Error rates
- Throughput
- Resource usage

**Environment:**
- Production-like
- Realistic data
- Monitor all services

---

### 5. How would you implement automated regression testing?

**Regression testing:** Ensure new changes don't break existing functionality.

**Implementation:**

**1. Comprehensive test suite:**
- Unit, integration, E2E
- Cover critical paths
- Automated execution

**2. CI/CD integration:**
- Run on every commit
- Block merge if failing
- Fast feedback

**3. Visual regression:**
- Screenshot comparison
- Percy, Chromatic
- UI changes detected

**4. API regression:**
- Request/response validation
- Performance benchmarks

**5. Test selection:**
- Run all for releases
- Run affected for PRs (test impact analysis)

**6. Monitoring:**
- Track test duration
- Flaky test detection
- Coverage trends

---

### 6-15. Advanced Quick Answers

**6. Testing eventual consistency:** Polling, wait strategies, verify convergence

**7. Testing database migrations:** Test up/down, rollback, data integrity

**8. Testing feature flags:** All combinations, gradual rollout simulation

**9. Load testing for 10,000+ users:** Distributed load generation, realistic scenarios

**10. Testing security vulnerabilities:** SAST/DAST, dependency scanning, pen testing

**11. Testing zero-downtime deployments:** Blue-green test, rolling update validation

**12. Testing third-party integrations:** VCR for recording, sandbox APIs, contract tests

**13. A/B testing infrastructure:** Feature flags, metrics tracking, statistical significance

**14. Testing data integrity:** Checksums, referential integrity, audit logs

**15. Testing strategy balancing speed and confidence:** Risk-based testing, parallelize, smart test selection

---

## Best Practices Summary

**All levels:**
- Write testable code
- Test early and often
- Automate everything
- Fast feedback loops
- Meaningful assertions
- Clear test names
- DRY in tests (but not too DRY)
- Monitor test health
- Balance coverage and speed
- Test what matters
