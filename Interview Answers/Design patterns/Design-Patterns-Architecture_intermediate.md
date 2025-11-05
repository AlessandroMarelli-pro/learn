# Design Patterns & Architecture - Intermediate Level Answers

---

## 1. Explain the difference between Strategy and State patterns

**Strategy pattern:**
- Multiple interchangeable algorithms
- Client chooses strategy
- Behavior varies by algorithm chosen
- Example: Different sorting algorithms, payment methods

**State pattern:**
- Object behavior changes based on internal state
- State transitions
- Object appears to change class
- Example: Order states (pending, paid, shipped)

**Key difference:** Strategy = choose algorithm, State = internal state determines behavior.

---

## 2. How would you implement the Decorator pattern in JavaScript/TypeScript?

**Decorator:** Adds functionality to object without modifying it.

**Purpose:**
- Add responsibilities dynamically
- Alternative to subclassing
- Flexible composition

**JavaScript implementation:**
- Wrapper objects
- Proxy pattern
- Function wrappers

**TypeScript decorators:**
- Class decorators
- Method decorators
- Property decorators
- Experimental feature

**Use cases:** Logging, validation, caching, authorization.

---

## 3. What is the Command pattern and when is it useful?

**Command:** Encapsulates request as object.

**Components:**
- **Command:** Interface with execute()
- **Concrete Command:** Implements execute()
- **Invoker:** Calls execute()
- **Receiver:** Performs actual action

**Benefits:**
- Decouple sender from receiver
- Queue/log commands
- Undo/redo functionality
- Macro commands

**Use cases:** Task queues, undo systems, schedulers, request logging.

---

## 4. Explain the Adapter pattern with a real-world use case

**Adapter:** Converts one interface to another expected by client.

**Purpose:**
- Make incompatible interfaces work together
- Wrap existing class with new interface
- Integration with third-party libraries

**Use case:**
- Adapt different payment gateways to common interface
- Unify multiple API responses
- Database adapters (MySQL, MongoDB, PostgreSQL)

**Analogy:** Power adapter for different countries.

---

## 5. How would you implement a Pub/Sub system?

**Pub/Sub (Publish/Subscribe):** Publishers emit events, subscribers receive them without direct connection.

**Components:**
- **Publisher:** Emits events
- **Subscriber:** Listens to events
- **Event Channel/Broker:** Routes messages

**Features:**
- Decoupling (publishers don't know subscribers)
- One-to-many
- Async communication

**Implementation:**
- In-memory: Event emitter
- Distributed: Redis pub/sub, RabbitMQ, Kafka

**Use cases:** Real-time notifications, microservices communication, event-driven architecture.

---

## 6. What is the difference between Active Record and Data Mapper patterns?

**Active Record:**
- Domain object contains data access logic
- Object knows how to save itself
- Tight coupling to database
- Simpler, less code
- Example: Mongoose models

**Data Mapper:**
- Separate mapper handles persistence
- Domain objects don't know database
- Loose coupling
- More flexible, testable
- Example: TypeORM with repositories

**Choose Active Record:** Simple apps, rapid development
**Choose Data Mapper:** Complex domains, strict separation.

---

## 7. Explain the Chain of Responsibility pattern

**Chain of Responsibility:** Pass request along chain of handlers until one handles it.

**Purpose:**
- Decouple sender from receiver
- Multiple handlers try in order
- Dynamic handler chain

**Structure:**
- Each handler has reference to next
- Can handle or pass to next

**Use cases:**
- Middleware chains (Express)
- Event bubbling (DOM)
- Authorization levels
- Logging levels

**Example:** Request → Auth → Validation → Rate Limit → Handler.

---

## 8. How would you structure a Node.js application using Clean Architecture?

**Clean Architecture layers (outer to inner):**

**1. Frameworks & Drivers (outermost):**
- Express, MongoDB
- External services

**2. Interface Adapters:**
- Controllers
- Presenters
- Gateways

**3. Use Cases:**
- Business logic
- Application-specific rules

**4. Entities (innermost):**
- Core business rules
- Domain models

**Dependency rule:** Inner layers don't depend on outer, only inward dependencies.

**Benefits:** Testable, independent of frameworks, flexible.

---

## 9. What is CQRS and when would you use it?

**CQRS (Command Query Responsibility Segregation):** Separate read and write models.

**Commands:**
- Modify state
- Return void or success
- Can be async

**Queries:**
- Read data
- No side effects
- Optimized for reading

**Benefits:**
- Optimize reads and writes separately
- Scale independently
- Different models for different needs

**Use cases:**
- Complex domains
- High read/write ratio difference
- Event sourcing
- Reporting requirements

**Trade-off:** Increased complexity.

---

## 10. Explain the Service Layer pattern

**Service Layer:** Defines application's boundary and operations.

**Purpose:**
- Encapsulate business logic
- Transaction boundaries
- Coordinate domain objects
- API for use cases

**Structure:**
- Controllers call services
- Services use repositories
- Services contain business logic

**Benefits:**
- Reusable business logic
- Testable
- Clear boundaries
- Thin controllers

**Example:** UserService.register(), OrderService.placeOrder().

---

## 11. How would you implement the Facade pattern in a complex API?

**Facade:** Simplified interface to complex subsystem.

**Purpose:**
- Hide complexity
- Provide simple API
- Reduce dependencies on subsystem

**Use case:**
- Complex third-party API
- Multiple microservices
- Legacy system integration

**Example:** PaymentFacade hides complexity of Stripe, PayPal, credit card validation, fraud detection.

**Benefits:** Easier to use, isolate changes, reduces coupling.

---

## 12. What is the Builder pattern and its benefits?

**Builder:** Construct complex object step by step.

**Purpose:**
- Create objects with many parameters
- Avoid telescoping constructors
- Make construction process clear
- Immutable objects

**Benefits:**
- Readable code
- Flexible construction
- Reusable builders
- Validation during construction

**Use case:** Building complex queries, HTTP requests, configurations.

---

## 13. Explain hexagonal architecture (Ports and Adapters)

**Hexagonal Architecture (Ports and Adapters):**

**Core (Domain):**
- Business logic
- No dependencies on external

**Ports:**
- Interfaces defining how to interact
- Input ports (use cases)
- Output ports (repositories)

**Adapters:**
- Implement ports
- Connect to external systems
- HTTP, database, queue, etc.

**Benefits:**
- Testable (mock adapters)
- Flexible (swap adapters)
- Independent of frameworks
- Clear boundaries

**Analogy:** Hexagon = core, each side = port, adapters plug in.

---

## 14. How would you implement feature flags in your architecture?

**Feature flags:** Toggle features on/off without deployment.

**Implementation:**
- Configuration file/database
- Evaluation service
- Check flag before feature code

**Types:**
- **Release flags:** Enable incomplete features
- **Experiment flags:** A/B testing
- **Ops flags:** Performance toggles
- **Permission flags:** User-specific features

**Best practices:**
- Clean up old flags
- Document flags
- Default safe state
- Monitoring

**Tools:** LaunchDarkly, Unleash, custom solution.

---

## 15. What are the benefits of using a layered architecture?

**Layered Architecture:** Application divided into horizontal layers.

**Common layers:**
1. Presentation (UI, Controllers)
2. Business Logic (Services)
3. Data Access (Repositories)
4. Database

**Benefits:**
- **Separation of concerns:** Each layer has specific role
- **Maintainability:** Changes isolated to layer
- **Testability:** Mock dependencies
- **Reusability:** Share business logic
- **Team organization:** Teams per layer

**Rules:**
- Layer only depends on layer below
- No skip-layer dependencies

**Trade-off:** Can add overhead, may not fit all domains.
