# Design Patterns & Architecture - Beginner Level Answers

---

## 1. What is the Singleton pattern and when would you use it?

**Singleton:** Ensures class has only one instance and provides global access point.

**Purpose:**
- Single shared instance across application
- Controlled access to resource

**Use cases:**
- Database connections
- Configuration objects
- Logging
- Caches

**Warning:** Can make testing difficult, creates global state.

---

## 2. Explain the Factory pattern with an example

**Factory:** Creates objects without specifying exact class.

**Purpose:**
- Encapsulate object creation
- Choose type at runtime
- Centralize creation logic

**Types:**
- **Simple Factory:** Single function/method
- **Factory Method:** Subclasses decide what to create
- **Abstract Factory:** Families of related objects

**Use case:** Creating different types of users, products, payments based on conditions.

---

## 3. What is the Observer pattern?

**Observer:** One-to-many dependency where observers are notified of subject changes.

**Components:**
- **Subject:** Maintains observers, notifies changes
- **Observers:** Subscribe to subject, react to changes

**Purpose:**
- Loose coupling
- Event-driven architecture
- Reactive systems

**Examples:** Event emitters, DOM events, Redux subscriptions, pub-sub systems.

---

## 4. Explain the Module pattern in JavaScript

**Module:** Uses closures to create private and public members.

**Purpose:**
- Encapsulation
- Private state
- Namespace management
- Prevent global pollution

**Implementation:**
- IIFE (Immediately Invoked Function Expression)
- Returns public API object
- Private variables in closure

**Modern:** ES6 modules with import/export.

---

## 5. What is the difference between composition and inheritance?

**Inheritance ("is-a" relationship):**
- Class extends another
- Inherits properties and methods
- Tight coupling
- Rigid hierarchy

**Composition ("has-a" relationship):**
- Object contains other objects
- Combines behaviors
- Loose coupling
- Flexible

**Best practice:** "Prefer composition over inheritance" - more flexible, easier to test.

---

## 6. What is dependency injection?

**Dependency Injection (DI):** Providing dependencies from outside rather than creating internally.

**Purpose:**
- Loose coupling
- Easier testing (mock dependencies)
- Flexibility
- Separation of concerns

**Types:**
- Constructor injection
- Property injection
- Method injection

**Example:** Pass database connection to service instead of creating inside.

---

## 7. Explain the MVC pattern

**MVC (Model-View-Controller):** Separates application into three components.

**Model:**
- Data and business logic
- Database interaction
- Independent of UI

**View:**
- Presentation layer
- Displays data
- User interface

**Controller:**
- Handles user input
- Updates model
- Selects view

**Benefits:** Separation of concerns, easier maintenance, parallel development.

---

## 8. What is the Repository pattern?

**Repository:** Abstraction layer between data access logic and business logic.

**Purpose:**
- Centralize data access
- Hide database details
- Easier to switch databases
- Testable (mock repository)

**Interface:** Define methods (find, save, delete, etc.)
**Implementation:** Actual database queries

**Use case:** In SaaS, repository per entity (UserRepository, ProductRepository).

---

## 9. What is the difference between a library and a framework?

**Library:**
- Collection of functions/utilities
- You call library code
- You control flow
- Examples: lodash, axios, moment

**Framework:**
- Complete structure for application
- Framework calls your code
- Inversion of control
- Examples: Express, NestJS, React

**Key difference:** Library = tool you use, Framework = structure you fit into.

---

## 10. Explain what RESTful API design means

**REST (Representational State Transfer):** Architectural style for APIs.

**Principles:**
- **Resources:** Nouns (users, posts)
- **HTTP methods:** GET, POST, PUT, DELETE
- **Stateless:** Each request independent
- **Uniform interface:** Consistent patterns
- **Client-server:** Separation

**RESTful conventions:**
- `GET /users` - List users
- `GET /users/123` - Get specific user
- `POST /users` - Create user
- `PUT /users/123` - Update user
- `DELETE /users/123` - Delete user

**Status codes:** 200 OK, 201 Created, 400 Bad Request, 404 Not Found, 500 Server Error.

---

## 11. What is separation of concerns?

**Separation of Concerns:** Different parts of code handle different responsibilities.

**Purpose:**
- Easier to understand
- Easier to maintain
- Easier to test
- Reduce coupling

**Examples:**
- MVC pattern
- Layers (presentation, business, data)
- Microservices (service per concern)

**Anti-pattern:** God object doing everything.

---

## 12. What is the DRY principle?

**DRY (Don't Repeat Yourself):** Every piece of knowledge should have single representation.

**Purpose:**
- Reduce duplication
- Single source of truth
- Easier maintenance
- Fewer bugs

**Application:**
- Extract reusable functions
- Use configuration files
- Template reuse

**Warning:** Balance with readability, don't over-abstract.

---

## 13. Explain the SOLID principles briefly

**S - Single Responsibility:**
- Class has one reason to change
- One responsibility

**O - Open/Closed:**
- Open for extension, closed for modification
- Add features without changing existing code

**L - Liskov Substitution:**
- Subtypes replaceable with base types
- Maintain contracts

**I - Interface Segregation:**
- Many specific interfaces better than one general
- Don't force unused methods

**D - Dependency Inversion:**
- Depend on abstractions, not concrete implementations
- High-level modules don't depend on low-level

---

## 14. What is the difference between synchronous and asynchronous patterns?

**Synchronous:**
- Operations execute sequentially
- Blocks until complete
- Simpler to understand
- Can freeze application

**Asynchronous:**
- Operations don't block
- Continue while waiting
- Better performance
- More complex (callbacks, promises)

**Node.js:** Built for async (non-blocking I/O).

**Use sync:** Startup scripts, CLI tools
**Use async:** Web servers, I/O operations.

---

## 15. What is middleware pattern in web applications?

**Middleware:** Functions that process requests before reaching final handler.

**Characteristics:**
- Chain of responsibility
- Each can modify request/response
- Can short-circuit (end request)
- Pass control with `next()`

**Use cases:**
- Authentication
- Logging
- Parsing request body
- Error handling
- CORS
- Rate limiting

**Order matters:** Middleware executes in registration order.
