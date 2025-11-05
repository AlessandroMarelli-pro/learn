# Technical Interview Questions - Senior Fullstack JavaScript Developer @ lempire

**Position Overview:** Senior Fullstack JavaScript Developer for a $40M ARR B2B SaaS company. Focus on Node.js, MongoDB, JavaScript/TypeScript, and full-stack development.

---

## JavaScript/TypeScript Fundamentals

### Beginner Level

1. What are the differences between `var`, `let`, and `const`?
2. Explain the concept of hoisting in JavaScript
3. What is the difference between `==` and `===`?
4. What are arrow functions and how do they differ from regular functions?
5. Explain the concept of closures with a simple example
6. What are Promises and how do they help with asynchronous code?
7. What is the event loop in JavaScript?
8. Explain the difference between `null` and `undefined`
9. What are template literals and when would you use them?
10. What is destructuring in JavaScript?
11. Explain the spread operator and rest parameters
12. What are the primitive data types in JavaScript?
13. What is the purpose of `async/await`?
14. How does `this` keyword work in JavaScript?
15. What is the difference between `.call()`, `.apply()`, and `.bind()`?

### Intermediate Level

1. Explain prototypal inheritance in JavaScript and how it differs from classical inheritance
2. What are JavaScript Symbols and when would you use them?
3. Explain the Module pattern and its benefits
4. What is memoization and how would you implement it?
5. Explain the difference between debouncing and throttling with use cases
6. What are JavaScript Proxies and Reflect API?
7. How does garbage collection work in JavaScript?
8. Explain the concept of currying and provide an example
9. What are Generators and Iterators in JavaScript?
10. How would you implement deep cloning of an object?
11. Explain the differences between TypeScript interfaces and types
12. What are TypeScript generics and why are they useful?
13. Explain covariance and contravariance in TypeScript
14. What is the purpose of `unknown` vs `any` in TypeScript?
15. How would you handle error boundaries in asynchronous code?

### Advanced Level

1. Explain the JavaScript memory model and how it affects performance optimization
2. How would you implement a custom event emitter from scratch?
3. Explain WeakMap and WeakSet and their practical use cases in preventing memory leaks
4. What are the performance implications of different data structures in V8?
5. Explain how you would implement a custom Promise implementation
6. What are the trade-offs between using TypeScript's strict mode features?
7. How would you design a type-safe event system in TypeScript?
8. Explain conditional types and mapped types in TypeScript with advanced examples
9. What are the internals of `async/await` (microtasks, macrotasks, event loop)?
10. How would you implement a cancellable Promise pattern?
11. Explain the concept of structural typing in TypeScript vs nominal typing
12. What are TypeScript decorators and how would you implement a custom one?
13. How would you optimize JavaScript code for JIT compilation?
14. Explain the concept of Tail Call Optimization and its support in JavaScript
15. How would you implement a reactive programming system from scratch?

---

## Node.js

### Beginner Level

1. What is Node.js and how does it differ from browser JavaScript?
2. Explain the Node.js event-driven architecture
3. What is `package.json` and what are its key properties?
4. What is the difference between `require()` and ES6 `import`?
5. What are streams in Node.js?
6. Explain the purpose of `process.env`
7. What is middleware in Express.js?
8. How do you handle environment variables in Node.js applications?
9. What is `npm` and what is `npx`?
10. Explain the purpose of the `cluster` module
11. What are the differences between development and production dependencies?
12. How do you read and write files in Node.js?
13. What is CORS and how do you handle it in Node.js?
14. Explain what callback hell is and how to avoid it
15. What is the purpose of `process.nextTick()`?

### Intermediate Level

1. Explain the difference between `process.nextTick()` and `setImmediate()`
2. How would you implement rate limiting in a Node.js API?
3. What are the different types of streams and when would you use each?
4. How do you handle uncaught exceptions and unhandled promise rejections?
5. Explain the Worker Threads API and when to use it
6. How would you implement authentication and authorization in a Node.js API?
7. What are the best practices for error handling in Express.js?
8. How do you implement pagination in a REST API?
9. Explain how to use Node.js child processes
10. What is the difference between `spawn`, `exec`, `execFile`, and `fork`?
11. How would you implement logging in a production Node.js application?
12. Explain how to handle file uploads in Node.js
13. What are the security best practices for Node.js applications?
14. How do you implement caching strategies in Node.js?
15. Explain how to profile and debug Node.js applications

### Advanced Level

1. How would you architect a Node.js application for horizontal scaling?
2. Explain the internals of Node.js event loop in detail (phases, queues)
3. How would you implement a custom stream in Node.js?
4. What strategies would you use to optimize Node.js application performance?
5. How do you handle memory leaks in Node.js applications?
6. Explain how to implement a microservices architecture with Node.js
7. How would you implement a job queue system from scratch?
8. What are the trade-offs between different Node.js frameworks (Express, Fastify, NestJS)?
9. How would you implement graceful shutdown in a Node.js application?
10. Explain how to use APM (Application Performance Monitoring) tools effectively
11. How would you implement a real-time notification system at scale?
12. What are the considerations for implementing WebSocket connections at scale?
13. How would you handle database connection pooling and optimization?
14. Explain how to implement distributed tracing in a Node.js microservices system
15. How would you design a Node.js application to handle 10,000+ concurrent connections?

---

## MongoDB & Database Design

### Beginner Level

1. What is MongoDB and how does it differ from SQL databases?
2. Explain the structure of a MongoDB document
3. What is a collection in MongoDB?
4. What are the basic CRUD operations in MongoDB?
5. What is the `_id` field in MongoDB?
6. Explain the difference between `find()` and `findOne()`
7. What is an index in MongoDB and why is it important?
8. How do you update a document in MongoDB?
9. What is the aggregation framework?
10. Explain the purpose of MongoDB drivers (like Mongoose)
11. What are the advantages of using MongoDB for a SaaS application?
12. How do you delete documents in MongoDB?
13. What is a replica set?
14. Explain what sharding is in MongoDB
15. What are the data types supported by MongoDB?

### Intermediate Level

1. Explain embedding vs referencing in MongoDB schema design
2. How would you design a schema for a multi-tenant SaaS application?
3. What are compound indexes and when should you use them?
4. Explain the aggregation pipeline and its common stages
5. How do you implement full-text search in MongoDB?
6. What are the best practices for indexing strategies?
7. Explain the difference between `update()`, `updateOne()`, and `updateMany()`
8. How do you implement pagination efficiently in MongoDB?
9. What is the purpose of `$lookup` in aggregation?
10. How would you handle database migrations in MongoDB?
11. Explain the concept of write concerns and read preferences
12. How do you implement transactions in MongoDB?
13. What are MongoDB validators and how do they work?
14. How would you optimize slow queries in MongoDB?
15. Explain the difference between Mongoose schemas and models

### Advanced Level

1. How would you design a MongoDB schema for optimal query performance in a B2B SaaS with multiple products?
2. Explain the trade-offs between different indexing strategies (B-tree, text, geospatial)
3. How would you implement a scalable audit log system in MongoDB?
4. What are the considerations for implementing multi-document transactions?
5. How would you optimize MongoDB performance for high write throughput?
6. Explain the internals of MongoDB's WiredTiger storage engine
7. How would you implement time-series data storage in MongoDB?
8. What strategies would you use for handling very large documents (near the 16MB limit)?
9. How would you design a MongoDB sharding key for a multi-tenant application?
10. Explain how to implement complex analytics queries efficiently
11. How would you handle MongoDB backup and disaster recovery strategies?
12. What are the performance implications of different schema design patterns?
13. How would you implement a search engine feature with MongoDB and potentially Elasticsearch?
14. Explain how to monitor and optimize MongoDB memory usage
15. How would you design a schema versioning strategy for a rapidly evolving SaaS product?

---

## Design Patterns & Architecture

### Beginner Level

1. What is the Singleton pattern and when would you use it?
2. Explain the Factory pattern with an example
3. What is the Observer pattern?
4. Explain the Module pattern in JavaScript
5. What is the difference between composition and inheritance?
6. What is dependency injection?
7. Explain the MVC pattern
8. What is the Repository pattern?
9. What is the difference between a library and a framework?
10. Explain what RESTful API design means
11. What is separation of concerns?
12. What is the DRY principle?
13. Explain the SOLID principles briefly
14. What is the difference between synchronous and asynchronous patterns?
15. What is middleware pattern in web applications?

### Intermediate Level

1. Explain the difference between Strategy and State patterns
2. How would you implement the Decorator pattern in JavaScript/TypeScript?
3. What is the Command pattern and when is it useful?
4. Explain the Adapter pattern with a real-world use case
5. How would you implement a Pub/Sub system?
6. What is the difference between Active Record and Data Mapper patterns?
7. Explain the Chain of Responsibility pattern
8. How would you structure a Node.js application using Clean Architecture?
9. What is CQRS and when would you use it?
10. Explain the Service Layer pattern
11. How would you implement the Facade pattern in a complex API?
12. What is the Builder pattern and its benefits?
13. Explain hexagonal architecture (Ports and Adapters)
14. How would you implement feature flags in your architecture?
15. What are the benefits of using a layered architecture?

### Advanced Level

1. How would you architect a multi-tenant B2B SaaS application for scalability and data isolation?
2. Explain Event Sourcing and its trade-offs in a SaaS context
3. How would you design a microservices architecture for a suite of related products?
4. What are the considerations for implementing domain-driven design (DDD)?
5. How would you handle service-to-service communication in a distributed system?
6. Explain the Saga pattern for distributed transactions
7. How would you implement a plugin architecture for extensibility?
8. What strategies would you use for API versioning in a fast-growing product?
9. How would you design a system for zero-downtime deployments?
10. Explain the CQRS + Event Sourcing pattern and its implementation challenges
11. How would you implement eventual consistency in a distributed system?
12. What are the trade-offs between monolith and microservices for a 130-person company?
13. How would you design a system to handle multiple products with shared infrastructure?
14. Explain how you would implement a circuit breaker pattern
15. How would you architect a system for multi-region deployment and data residency?

---

## System Design & Scalability

### Beginner Level

1. What is horizontal vs vertical scaling?
2. Explain what load balancing is
3. What is caching and why is it important?
4. What is the difference between stateful and stateless applications?
5. Explain what an API gateway is
6. What is rate limiting?
7. What is the purpose of a reverse proxy?
8. Explain what CDN is and its benefits
9. What are webhooks?
10. What is the difference between authentication and authorization?
11. Explain what session management is
12. What is the purpose of logging?
13. What are health checks in web applications?
14. Explain what a message queue is
15. What is the difference between SQL and NoSQL databases?

### Intermediate Level

1. How would you implement a caching strategy for a SaaS application?
2. Explain different types of caching (in-memory, distributed, HTTP)
3. How would you design a rate limiting system?
4. What are the considerations for implementing a job queue?
5. How would you implement real-time notifications at scale?
6. Explain the CAP theorem and its implications
7. How would you design a logging and monitoring system?
8. What strategies would you use for handling peak traffic?
9. How would you implement search functionality in a large dataset?
10. Explain the difference between optimistic and pessimistic locking
11. How would you design a system for handling file uploads and storage?
12. What are the considerations for implementing webhooks?
13. How would you implement a multi-tenant database strategy?
14. Explain the benefits and drawbacks of serverless architecture
15. How would you design an API for third-party integrations?

### Advanced Level

1. How would you design a system to handle $40M ARR scale with 130 employees?
2. Explain how you would architect a globally distributed application
3. How would you design a system for 99.99% uptime?
4. What strategies would you use for database scaling (read replicas, sharding, partitioning)?
5. How would you implement a real-time analytics system?
6. Explain how you would design a system to handle millions of webhook deliveries
7. How would you architect a system for data encryption at rest and in transit?
8. What are the considerations for implementing GDPR compliance in a global SaaS?
9. How would you design a disaster recovery and backup strategy?
10. Explain how you would implement a feature rollout system with gradual deployment
11. How would you handle data consistency across multiple products in the suite?
12. What strategies would you use for optimizing database costs at scale?
13. How would you design a system to support multiple API versions simultaneously?
14. Explain how you would implement observability (logs, metrics, traces) at scale
15. How would you architect a system to support white-labeling for enterprise customers?

---

## Testing & Quality

### Beginner Level

1. What is the difference between unit tests, integration tests, and e2e tests?
2. What is Test-Driven Development (TDD)?
3. What testing frameworks have you used in JavaScript?
4. How do you mock dependencies in tests?
5. What is code coverage and why does it matter?
6. Explain what assertions are in testing
7. What is the purpose of test fixtures?
8. How do you test async code in JavaScript?
9. What is the difference between stub and spy?
10. What is continuous integration (CI)?
11. How do you test REST APIs?
12. What is the purpose of linting?
13. What is the testing pyramid?
14. How do you handle test data?
15. What are snapshot tests?

### Intermediate Level

1. How would you structure tests in a Node.js application?
2. Explain the difference between mocks, stubs, and spies
3. How do you test database interactions?
4. What strategies do you use for testing error scenarios?
5. How would you implement contract testing for APIs?
6. Explain how to test middleware in Express
7. How do you test authentication and authorization?
8. What is mutation testing?
9. How would you test WebSocket connections?
10. Explain how to test event-driven code
11. How do you handle flaky tests?
12. What is the purpose of integration testing in a microservices architecture?
13. How would you test file uploads and processing?
14. Explain property-based testing
15. How do you implement end-to-end tests for a SaaS application?

### Advanced Level

1. How would you design a testing strategy for a suite of B2B SaaS products?
2. Explain how you would implement chaos engineering practices
3. How would you test multi-tenant isolation?
4. What strategies would you use for performance testing at scale?
5. How would you implement automated regression testing?
6. Explain how to test eventual consistency in distributed systems
7. How would you test database migrations without downtime?
8. What strategies would you use for testing feature flags?
9. How would you implement load testing for 10,000+ concurrent users?
10. Explain how to test security vulnerabilities systematically
11. How would you design a testing strategy for zero-downtime deployments?
12. What approaches would you take for testing third-party integrations?
13. How would you implement A/B testing infrastructure?
14. Explain how to test data integrity across system boundaries
15. How would you design a testing strategy that balances speed and confidence?

---

## Behavioral & Problem-Solving Questions

### Leadership & Ownership

1. Describe a time when you took full ownership of a complex feature from design to deployment
2. How do you approach working independently on a project with minimal guidance?
3. Describe your experience mentoring other developers or leading technical initiatives
4. Tell me about a major refactoring project you led and the impact it had
5. Describe a time when you introduced a new technology or practice to your team

### Remote Work & Communication

6. What strategies do you use to stay productive in a fully remote environment?
7. How do you ensure effective communication in a remote team using tools like Slack, Notion, and Gather?
8. How do you handle working across different time zones?

### Technical Problem-Solving

9. How would you approach helping customer support resolve technical issues?
10. How do you prioritize and manage working on multiple projects simultaneously?
11. Describe a time when you identified and fixed a significant performance bottleneck
12. Walk me through your approach to debugging a production issue
13. How do you balance feature development with technical debt reduction?

### Learning & Adaptability

14. How do you stay current with JavaScript/Node.js ecosystem developments?
15. What are your non-negotiables when it comes to code quality?

---

## Practical Scenarios for lempire

1. A customer reports that API responses are slow during peak hours. How would you investigate and resolve this?
2. You need to add a new feature that requires changes across frontend, backend, and database. Walk me through your approach
3. How would you implement a new integration with a third-party service for one of the products?
4. The team wants to optimize MongoDB queries that are causing performance issues. What's your approach?
5. You discover a security vulnerability in production code. What steps do you take?
6. How would you design a feature flag system to gradually roll out new features?
7. A new product in the suite needs to share authentication with existing products. How would you architect this?
8. You need to migrate data from an old schema to a new one without downtime. What's your strategy?
9. How would you implement analytics tracking across multiple products in the suite?
10. Customer support is overwhelmed with technical questions about API limits. How would you address this technically and systematically?

---

## Interview Preparation Tips

**What lempire is Looking For:**
- **Autonomy**: Ability to work independently with minimal guidance
- **B2B SaaS Experience**: Understanding of multi-tenant, scalable systems
- **Passion for Coding**: Personal projects and continuous learning
- **Node.js & MongoDB Expertise**: Deep knowledge, not just surface-level
- **Remote Work Proficiency**: Comfortable with Slack, Notion, Gather
- **Ownership**: Taking responsibility for complex projects end-to-end
- **Technical Leadership**: Ability to set standards and mentor others
- **Adaptability**: Wearing multiple hats (frontend, backend, support, optimization)
- **Speed & Quality Balance**: Delivering fast without compromising standards
- **Scalability Mindset**: Thinking about $40M ARR scale challenges
