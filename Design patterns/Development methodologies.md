I'll create a comprehensive guide covering DDD, TDD, and related development methodologies.

# Software Development Methodologies - Interview Preparation

## Domain-Driven Design (DDD)

### Simple Explanation

DDD is an approach to software development that focuses on understanding and modeling the business domain (the real-world problem you're solving) as the foundation of your software design.

Instead of starting with databases or technical architecture, you start by deeply understanding the business, its rules, and its language. Then you build software that reflects that understanding.

**Example:** If you're building banking software, you don't just think about "saving data to a database." You think about "accounts," "transactions," "overdraft rules," "interest calculations" - the actual banking concepts that domain experts (bankers) understand.

### Technical Explanation

DDD is a software design philosophy introduced by Eric Evans in 2003. It emphasizes collaboration between technical experts and domain experts to create a software model that accurately reflects business reality.

**Core concepts:**

#### 1. Ubiquitous Language

A shared vocabulary between developers and domain experts that appears in both conversations and code.

```typescript
// BAD - Technical jargon
class UserData {
  processRecord() {}
}

// GOOD - Ubiquitous Language
class Order {
  placeOrder() {}
  cancelOrder() {}
  confirmPayment() {}
}
```

#### 2. Bounded Context

A boundary within which a particular domain model is defined and applicable. Different contexts can have different models for the same concept.

```typescript
// In Sales Context
class Customer {
  customerId: string;
  creditLimit: Money;
  outstandingOrders: Order[];
}

// In Support Context
class Customer {
  customerId: string;
  supportTickets: Ticket[];
  satisfactionRating: number;
}
```

#### 3. Entities

Objects with a distinct identity that persists over time.

```typescript
class Order {
  constructor(
    public readonly orderId: string, // Identity
    public customerId: string,
    public items: OrderItem[],
    public status: OrderStatus,
  ) {}

  // Two orders with same data but different IDs are different
}
```

#### 4. Value Objects

Objects defined by their attributes, not identity. Immutable.

```typescript
class Money {
  constructor(
    public readonly amount: number,
    public readonly currency: string,
  ) {}

  add(other: Money): Money {
    if (this.currency !== other.currency) {
      throw new Error('Cannot add different currencies');
    }
    return new Money(this.amount + other.amount, this.currency);
  }

  // Two Money objects with same amount and currency are equal
}

class Address {
  constructor(
    public readonly street: string,
    public readonly city: string,
    public readonly zipCode: string,
  ) {}
}
```

#### 5. Aggregates

A cluster of entities and value objects with a root entity (Aggregate Root) that controls access.

```typescript
// Order is the Aggregate Root
class Order {
  private orderId: string;
  private customerId: string;
  private items: OrderItem[] = []; // Internal entities
  private status: OrderStatus;

  // Only way to modify items is through the aggregate root
  addItem(product: Product, quantity: number): void {
    if (this.status !== OrderStatus.DRAFT) {
      throw new Error('Cannot modify confirmed order');
    }
    this.items.push(new OrderItem(product, quantity));
  }

  // Business rules enforced at aggregate boundary
  confirmOrder(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot confirm empty order');
    }
    this.status = OrderStatus.CONFIRMED;
  }
}

// OrderItem is only accessible through Order
class OrderItem {
  constructor(public product: Product, public quantity: number) {}
}
```

#### 6. Repositories

Abstracts data access, providing collection-like interface for aggregates.

```typescript
interface OrderRepository {
  findById(orderId: string): Promise<Order | null>;
  save(order: Order): Promise<void>;
  findByCustomerId(customerId: string): Promise<Order[]>;
}

class InMemoryOrderRepository implements OrderRepository {
  private orders: Map<string, Order> = new Map();

  async findById(orderId: string): Promise<Order | null> {
    return this.orders.get(orderId) || null;
  }

  async save(order: Order): Promise<void> {
    this.orders.set(order.orderId, order);
  }

  async findByCustomerId(customerId: string): Promise<Order[]> {
    return Array.from(this.orders.values()).filter(
      (order) => order.customerId === customerId,
    );
  }
}
```

#### 7. Domain Services

Operations that don't naturally fit in entities or value objects.

```typescript
class PricingService {
  calculateOrderTotal(order: Order, customer: Customer): Money {
    let total = order.items.reduce(
      (sum, item) => sum.add(item.price.multiply(item.quantity)),
      Money.zero('USD'),
    );

    // Apply customer-specific discount
    if (customer.isPremium()) {
      total = total.multiply(0.9); // 10% discount
    }

    return total;
  }
}
```

#### 8. Domain Events

Something significant that happened in the domain.

```typescript
class OrderPlacedEvent {
  constructor(
    public readonly orderId: string,
    public readonly customerId: string,
    public readonly totalAmount: Money,
    public readonly occurredAt: Date,
  ) {}
}

class Order {
  private events: DomainEvent[] = [];

  placeOrder(): void {
    // Business logic
    this.status = OrderStatus.PLACED;

    // Record event
    this.events.push(
      new OrderPlacedEvent(
        this.orderId,
        this.customerId,
        this.total,
        new Date(),
      ),
    );
  }

  getEvents(): DomainEvent[] {
    return this.events;
  }
}
```

**Layered Architecture in DDD:**

```
┌─────────────────────────────────┐
│   Presentation Layer            │  (Controllers, Views)
├─────────────────────────────────┤
│   Application Layer             │  (Use Cases, Application Services)
├─────────────────────────────────┤
│   Domain Layer                  │  (Entities, Value Objects, Domain Services)
├─────────────────────────────────┤
│   Infrastructure Layer          │  (Repositories, External Services)
└─────────────────────────────────┘
```

**When to use DDD:**

- Complex business logic
- Long-term projects
- Need close collaboration with domain experts
- Core business competitive advantage

**When NOT to use DDD:**

- Simple CRUD applications
- Short-term projects
- Trivial business logic
- Technical/infrastructure problems

---

## Test-Driven Development (TDD)

### Simple Explanation

TDD is a development approach where you write tests BEFORE writing the actual code. It's like planning your destination before starting a journey.

**The cycle:**

1. **Red:** Write a failing test for the feature you want
2. **Green:** Write just enough code to make the test pass
3. **Refactor:** Clean up the code while keeping tests green

**Example:** You want to add two numbers.

1. Write test: `expect(add(2, 3)).toBe(5)` - it fails (no `add` function yet)
2. Write code: `function add(a, b) { return a + b }` - test passes
3. Refactor if needed

### Technical Explanation

TDD is a software development methodology where tests drive the design and development of software. Created by Kent Beck as part of Extreme Programming (XP).

**The TDD Cycle (Red-Green-Refactor):**

```typescript
// 1. RED - Write failing test first
describe('Calculator', () => {
  it('should add two numbers', () => {
    const calculator = new Calculator();
    expect(calculator.add(2, 3)).toBe(5);
  });
});

// Test fails - Calculator doesn't exist yet

// 2. GREEN - Write minimal code to pass
class Calculator {
  add(a: number, b: number): number {
    return a + b;
  }
}

// Test passes!

// 3. REFACTOR - Improve code quality
// (In this case, code is already clean)
```

**TDD Rules (by Uncle Bob):**

1. Don't write production code unless it's to make a failing test pass
2. Don't write more of a test than is sufficient to fail
3. Don't write more production code than necessary to pass the failing test

**Example - Building a Shopping Cart:**

```typescript
// Step 1: Write test for empty cart
describe('ShoppingCart', () => {
  it('should start empty', () => {
    const cart = new ShoppingCart();
    expect(cart.getItemCount()).toBe(0);
  });
});

// Step 2: Implement minimal code
class ShoppingCart {
  private items: any[] = [];

  getItemCount(): number {
    return this.items.length;
  }
}

// Step 3: Test adding items
describe('ShoppingCart', () => {
  it('should add items to cart', () => {
    const cart = new ShoppingCart();
    cart.addItem({ name: 'Laptop', price: 1000 });
    expect(cart.getItemCount()).toBe(1);
  });
});

// Step 4: Implement
class ShoppingCart {
  private items: CartItem[] = [];

  addItem(item: CartItem): void {
    this.items.push(item);
  }

  getItemCount(): number {
    return this.items.length;
  }
}

// Step 5: Test calculating total
describe('ShoppingCart', () => {
  it('should calculate total price', () => {
    const cart = new ShoppingCart();
    cart.addItem({ name: 'Laptop', price: 1000 });
    cart.addItem({ name: 'Mouse', price: 50 });
    expect(cart.getTotal()).toBe(1050);
  });
});

// Step 6: Implement
class ShoppingCart {
  private items: CartItem[] = [];

  addItem(item: CartItem): void {
    this.items.push(item);
  }

  getItemCount(): number {
    return this.items.length;
  }

  getTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}

interface CartItem {
  name: string;
  price: number;
}
```

**Types of Tests in TDD:**

#### 1. Unit Tests

Test individual units (functions, classes) in isolation.

```typescript
describe('User', () => {
  it('should validate email format', () => {
    const user = new User('john@example.com', 'password123');
    expect(user.isEmailValid()).toBe(true);
  });

  it('should reject invalid email', () => {
    const user = new User('invalid-email', 'password123');
    expect(user.isEmailValid()).toBe(false);
  });
});
```

#### 2. Integration Tests

Test how components work together.

```typescript
describe('OrderService', () => {
  it('should create order and update inventory', async () => {
    const orderService = new OrderService(
      new OrderRepository(),
      new InventoryService(),
    );

    const order = await orderService.placeOrder(customerId, items);

    expect(order.status).toBe('CONFIRMED');
    expect(await inventoryService.getStock(productId)).toBe(
      initialStock - quantity,
    );
  });
});
```

#### 3. End-to-End Tests

Test complete user workflows.

```typescript
describe('Checkout Flow', () => {
  it('should complete full checkout process', async () => {
    await page.goto('/products');
    await page.click('#add-to-cart-laptop');
    await page.goto('/cart');
    await page.click('#checkout');
    await page.fill('#card-number', '4242424242424242');
    await page.click('#place-order');

    expect(await page.textContent('.confirmation')).toContain(
      'Order placed successfully',
    );
  });
});
```

**Benefits of TDD:**

- **Better design:** Forces you to think about interfaces
- **Living documentation:** Tests document how code should work
- **Confidence:** Refactor without fear
- **Fewer bugs:** Catches issues early
- **Test coverage:** High coverage by design

**Challenges:**

- Initial slowdown (writing tests first)
- Learning curve
- Difficult for legacy code
- Can lead to over-testing trivial code

**Best Practices:**

- Keep tests simple and readable
- Test behavior, not implementation
- One assertion per test (ideally)
- Use descriptive test names
- Follow AAA pattern: Arrange, Act, Assert

```typescript
describe('BankAccount', () => {
  it('should allow withdrawal when sufficient balance', () => {
    // Arrange
    const account = new BankAccount(100);

    // Act
    const result = account.withdraw(50);

    // Assert
    expect(result.success).toBe(true);
    expect(account.getBalance()).toBe(50);
  });
});
```

---

## Behavior-Driven Development (BDD)

### Simple Explanation

BDD extends TDD by writing tests in plain language that business people can understand. Tests describe behaviors from the user's perspective.

Instead of: "Test that the add method returns the sum"
BDD says: "Given a calculator, when I add 2 and 3, then I should get 5"

### Technical Explanation

BDD focuses on the behavior of the application from the end user's perspective. Tests are written in a Given-When-Then format using natural language.

**Example with Jest/Cucumber:**

```typescript
// Feature file (Gherkin syntax)
Feature: Shopping Cart
  As a customer
  I want to add items to my cart
  So that I can purchase multiple items

  Scenario: Adding items to cart
    Given I have an empty shopping cart
    When I add a laptop costing $1000
    And I add a mouse costing $50
    Then my cart should contain 2 items
    And the total should be $1050

// Step definitions
import { Given, When, Then } from '@cucumber/cucumber';

let cart: ShoppingCart;

Given('I have an empty shopping cart', () => {
  cart = new ShoppingCart();
});

When('I add a laptop costing ${int}', (price: number) => {
  cart.addItem({ name: 'Laptop', price });
});

When('I add a mouse costing ${int}', (price: number) => {
  cart.addItem({ name: 'Mouse', price });
});

Then('my cart should contain {int} items', (count: number) => {
  expect(cart.getItemCount()).toBe(count);
});

Then('the total should be ${int}', (total: number) => {
  expect(cart.getTotal()).toBe(total);
});
```

**Using Jest (JavaScript style BDD):**

```typescript
describe('Shopping Cart', () => {
  describe('when empty', () => {
    it('should have 0 items', () => {
      const cart = new ShoppingCart();
      expect(cart.getItemCount()).toBe(0);
    });

    it('should have total of 0', () => {
      const cart = new ShoppingCart();
      expect(cart.getTotal()).toBe(0);
    });
  });

  describe('when adding items', () => {
    let cart: ShoppingCart;

    beforeEach(() => {
      cart = new ShoppingCart();
    });

    it('should increase item count', () => {
      cart.addItem({ name: 'Laptop', price: 1000 });
      expect(cart.getItemCount()).toBe(1);
    });

    it('should update total price', () => {
      cart.addItem({ name: 'Laptop', price: 1000 });
      expect(cart.getTotal()).toBe(1000);
    });
  });
});
```

**Benefits:**

- Better collaboration with non-technical stakeholders
- Tests serve as living documentation
- Focus on user value
- Clearer requirements

---

## Other Related Concepts

### 1. Acceptance Test-Driven Development (ATDD)

#### Simple Explanation

ATDD is like BDD but focuses on writing acceptance criteria before development. Team members (developers, testers, customers) collaborate to define acceptance tests.

#### Technical Explanation

ATDD involves creating tests from acceptance criteria during or before implementation.

```typescript
// Acceptance Criteria:
// - User can register with email and password
// - Password must be at least 8 characters
// - Email must be valid format
// - User receives confirmation email

describe('User Registration', () => {
  it('should successfully register user with valid credentials', async () => {
    const userService = new UserService();
    const result = await userService.register(
      'john@example.com',
      'password123',
    );

    expect(result.success).toBe(true);
    expect(result.user.email).toBe('john@example.com');
  });

  it('should reject short passwords', async () => {
    const userService = new UserService();
    const result = await userService.register('john@example.com', 'pass');

    expect(result.success).toBe(false);
    expect(result.error).toContain('Password must be at least 8 characters');
  });

  it('should reject invalid email format', async () => {
    const userService = new UserService();
    const result = await userService.register('invalid-email', 'password123');

    expect(result.success).toBe(false);
    expect(result.error).toContain('Invalid email format');
  });

  it('should send confirmation email', async () => {
    const emailService = mock(EmailService);
    const userService = new UserService(emailService);

    await userService.register('john@example.com', 'password123');

    expect(emailService.send).toHaveBeenCalledWith(
      expect.objectContaining({
        to: 'john@example.com',
        subject: 'Confirm your registration',
      }),
    );
  });
});
```

---

### 2. Event-Driven Architecture (EDA)

#### Simple Explanation

EDA is an architectural pattern where components communicate by producing and consuming events. It's like a notification system - when something happens, all interested parties are notified.

#### Technical Explanation

Systems react to events (significant state changes) rather than direct calls.

```typescript
// Event
class OrderPlacedEvent {
  constructor(
    public orderId: string,
    public customerId: string,
    public total: number,
    public timestamp: Date,
  ) {}
}

// Event Bus
class EventBus {
  private handlers: Map<string, Function[]> = new Map();

  subscribe(eventType: string, handler: Function): void {
    if (!this.handlers.has(eventType)) {
      this.handlers.set(eventType, []);
    }
    this.handlers.get(eventType)!.push(handler);
  }

  publish(event: any): void {
    const eventType = event.constructor.name;
    const handlers = this.handlers.get(eventType) || [];
    handlers.forEach((handler) => handler(event));
  }
}

// Subscribers
class InventoryService {
  constructor(private eventBus: EventBus) {
    eventBus.subscribe('OrderPlacedEvent', this.handleOrderPlaced.bind(this));
  }

  private handleOrderPlaced(event: OrderPlacedEvent): void {
    console.log(`Reserving inventory for order ${event.orderId}`);
    // Reserve inventory logic
  }
}

class EmailService {
  constructor(private eventBus: EventBus) {
    eventBus.subscribe('OrderPlacedEvent', this.handleOrderPlaced.bind(this));
  }

  private handleOrderPlaced(event: OrderPlacedEvent): void {
    console.log(`Sending confirmation email for order ${event.orderId}`);
    // Send email logic
  }
}

// Publisher
class OrderService {
  constructor(private eventBus: EventBus) {}

  placeOrder(customerId: string, items: any[]): void {
    const orderId = generateId();
    // Create order logic...

    // Publish event
    this.eventBus.publish(
      new OrderPlacedEvent(
        orderId,
        customerId,
        calculateTotal(items),
        new Date(),
      ),
    );
  }
}

// Usage
const eventBus = new EventBus();
const inventoryService = new InventoryService(eventBus);
const emailService = new EmailService(eventBus);
const orderService = new OrderService(eventBus);

orderService.placeOrder('customer-123', [
  /* items */
]);
// Both inventory and email services are automatically notified
```

---

### 3. Event Sourcing

#### Simple Explanation

Instead of storing just the current state, Event Sourcing stores all changes (events) that led to that state. It's like a bank statement - instead of just showing your current balance, it shows every transaction.

#### Technical Explanation

The state of a system is determined by replaying events.

```typescript
// Events
abstract class Event {
  constructor(public timestamp: Date = new Date()) {}
}

class AccountCreatedEvent extends Event {
  constructor(public accountId: string, public initialBalance: number) {
    super();
  }
}

class MoneyDepositedEvent extends Event {
  constructor(public accountId: string, public amount: number) {
    super();
  }
}

class MoneyWithdrawnEvent extends Event {
  constructor(public accountId: string, public amount: number) {
    super();
  }
}

// Event Store
class EventStore {
  private events: Map<string, Event[]> = new Map();

  append(aggregateId: string, event: Event): void {
    if (!this.events.has(aggregateId)) {
      this.events.set(aggregateId, []);
    }
    this.events.get(aggregateId)!.push(event);
  }

  getEvents(aggregateId: string): Event[] {
    return this.events.get(aggregateId) || [];
  }
}

// Aggregate that uses Event Sourcing
class BankAccount {
  private accountId: string;
  private balance: number = 0;
  private version: number = 0;

  constructor(accountId: string) {
    this.accountId = accountId;
  }

  // Replay events to rebuild state
  static fromEvents(accountId: string, events: Event[]): BankAccount {
    const account = new BankAccount(accountId);
    events.forEach((event) => account.apply(event));
    return account;
  }

  // Apply event to change state
  private apply(event: Event): void {
    if (event instanceof AccountCreatedEvent) {
      this.balance = event.initialBalance;
    } else if (event instanceof MoneyDepositedEvent) {
      this.balance += event.amount;
    } else if (event instanceof MoneyWithdrawnEvent) {
      this.balance -= event.amount;
    }
    this.version++;
  }

  // Commands that generate events
  createAccount(initialBalance: number, eventStore: EventStore): void {
    const event = new AccountCreatedEvent(this.accountId, initialBalance);
    this.apply(event);
    eventStore.append(this.accountId, event);
  }

  deposit(amount: number, eventStore: EventStore): void {
    const event = new MoneyDepositedEvent(this.accountId, amount);
    this.apply(event);
    eventStore.append(this.accountId, event);
  }

  withdraw(amount: number, eventStore: EventStore): void {
    if (amount > this.balance) {
      throw new Error('Insufficient funds');
    }
    const event = new MoneyWithdrawnEvent(this.accountId, amount);
    this.apply(event);
    eventStore.append(this.accountId, event);
  }

  getBalance(): number {
    return this.balance;
  }
}

// Usage
const eventStore = new EventStore();
const account = new BankAccount('acc-123');

account.createAccount(1000, eventStore);
account.deposit(500, eventStore);
account.withdraw(200, eventStore);

console.log('Current balance:', account.getBalance()); // 1300

// Rebuild state from events
const events = eventStore.getEvents('acc-123');
const rebuiltAccount = BankAccount.fromEvents('acc-123', events);
console.log('Rebuilt balance:', rebuiltAccount.getBalance()); // 1300

// View event history
events.forEach((event) => {
  console.log(event);
});
```

**Benefits:**

- Complete audit trail
- Time travel (view state at any point)
- Event replay for debugging
- Easy to add new projections

---

### 4. CQRS (Command Query Responsibility Segregation)

#### Simple Explanation

CQRS separates reading data (queries) from writing data (commands). It's like having different doors for entering and exiting a building - optimized for different purposes.

#### Technical Explanation

Separate models for updates (commands) and reads (queries).

```typescript
// Command side (Write Model)
interface Command {
  execute(): void;
}

class CreateOrderCommand implements Command {
  constructor(
    private orderId: string,
    private customerId: string,
    private items: OrderItem[],
  ) {}

  execute(): void {
    // Validate business rules
    // Create order
    // Save to write database
    console.log('Order created');
  }
}

class CancelOrderCommand implements Command {
  constructor(private orderId: string) {}

  execute(): void {
    // Validate cancellation rules
    // Update order status
    // Save to write database
    console.log('Order cancelled');
  }
}

// Query side (Read Model)
interface OrderReadModel {
  orderId: string;
  customerName: string;
  totalAmount: number;
  status: string;
  items: Array<{
    productName: string;
    quantity: number;
    price: number;
  }>;
}

class OrderQueryService {
  // Optimized read database (denormalized)
  private readDatabase: Map<string, OrderReadModel> = new Map();

  getOrderById(orderId: string): OrderReadModel | null {
    return this.readDatabase.get(orderId) || null;
  }

  getOrdersByCustomer(customerId: string): OrderReadModel[] {
    return Array.from(this.readDatabase.values()).filter(
      (order) => order.customerId === customerId,
    );
  }

  getRecentOrders(limit: number): OrderReadModel[] {
    return Array.from(this.readDatabase.values())
      .sort((a, b) => b.createdAt - a.createdAt)
      .slice(0, limit);
  }
}

// Command Handler
class CommandHandler {
  handle(command: Command): void {
    command.execute();
    // Update read model asynchronously
    this.updateReadModel(command);
  }

  private updateReadModel(command: Command): void {
    // Project changes to read model
    // This can be async
  }
}

// Usage
const commandHandler = new CommandHandler();
const queryService = new OrderQueryService();

// Write
commandHandler.handle(new CreateOrderCommand('order-1', 'customer-1', items));

// Read (from different model)
const order = queryService.getOrderById('order-1');
const customerOrders = queryService.getOrdersByCustomer('customer-1');
```

**Benefits:**

- Optimized read and write models
- Scalability (scale reads and writes independently)
- Flexibility (different databases for different needs)
- Better performance

**Challenges:**

- Increased complexity
- Eventual consistency
- More infrastructure

---

### 5. Hexagonal Architecture (Ports and Adapters)

#### Simple Explanation

Hexagonal Architecture isolates your core business logic from external concerns (databases, APIs, UI). The business logic is at the center, and everything else is pluggable around it.

It's like a board game - the rules are at the center, and you can play it on a physical board, computer, or mobile app (different adapters).

#### Technical Explanation

```typescript
// Domain (Core - Center of Hexagon)
class Order {
  constructor(
    public orderId: string,
    public customerId: string,
    public items: OrderItem[],
    public status: OrderStatus,
  ) {}

  confirm(): void {
    if (this.items.length === 0) {
      throw new Error('Cannot confirm empty order');
    }
    this.status = OrderStatus.CONFIRMED;
  }
}

// Ports (Interfaces)
interface OrderRepository {
  save(order: Order): Promise<void>;
  findById(orderId: string): Promise<Order | null>;
}

interface PaymentGateway {
  processPayment(amount: number, customerId: string): Promise<boolean>;
}

interface NotificationService {
  sendOrderConfirmation(order: Order): Promise<void>;
}

// Application Service (Use Case)
class OrderService {
  constructor(
    private orderRepository: OrderRepository,
    private paymentGateway: PaymentGateway,
    private notificationService: NotificationService,
  ) {}

  async placeOrder(customerId: string, items: OrderItem[]): Promise<Order> {
    // Business logic
    const order = new Order(
      generateId(),
      customerId,
      items,
      OrderStatus.PENDING,
    );

    const totalAmount = items.reduce((sum, item) => sum + item.price, 0);
    const paymentSuccess = await this.paymentGateway.processPayment(
      totalAmount,
      customerId,
    );

    if (!paymentSuccess) {
      throw new Error('Payment failed');
    }

    order.confirm();
    await this.orderRepository.save(order);
    await this.notificationService.sendOrderConfirmation(order);

    return order;
  }
}

// Adapters (Implementations)
class PostgresOrderRepository implements OrderRepository {
  async save(order: Order): Promise<void> {
    console.log('Saving to PostgreSQL...');
    // SQL implementation
  }

  async findById(orderId: string): Promise<Order | null> {
    console.log('Fetching from PostgreSQL...');
    // SQL implementation
    return null;
  }
}

class StripePaymentGateway implements PaymentGateway {
  async processPayment(amount: number, customerId: string): Promise<boolean> {
    console.log('Processing payment via Stripe...');
    // Stripe API call
    return true;
  }
}

class EmailNotificationService implements NotificationService {
  async sendOrderConfirmation(order: Order): Promise<void> {
    console.log('Sending email notification...');
    // Email sending logic
  }
}

// Wiring (Dependency Injection)
const orderRepository = new PostgresOrderRepository();
const paymentGateway = new StripePaymentGateway();
const notificationService = new EmailNotificationService();

const orderService = new OrderService(
  orderRepository,
  paymentGateway,
  notificationService,
);

// Easy to swap implementations for testing
class InMemoryOrderRepository implements OrderRepository {
  private orders: Map<string, Order> = new Map();

  async save(order: Order): Promise<void> {
    this.orders.set(order.orderId, order);
  }

  async findById(orderId: string): Promise<Order | null> {
    return this.orders.get(orderId) || null;
  }
}

// For tests
const testOrderService = new OrderService(
  new InMemoryOrderRepository(), // Different adapter
  new MockPaymentGateway(),
  new MockNotificationService(),
);
```

**Structure:**

```
         ┌────────────────────────┐
         │    HTTP Adapter        │
         │    (REST API)          │
         └───────────┬────────────┘
                     │
         ┌───────────▼────────────┐
         │                        │
    ┌────┤   Application Core    ├────┐
    │    │   (Business Logic)    │    │
    │    │                        │    │
    │    └────────────────────────┘    │
    │                                   │
┌───▼────────┐                  ┌──────▼────┐
│ Database   │                  │  Payment  │
│ Adapter    │                  │  Adapter  │
│ (PostgreSQL)│                 │ (Stripe)  │
└────────────┘                  └───────────┘
```

**Benefits:**

- Testable (swap implementations easily)
- Technology independent
- Framework independent
- Clear boundaries

---

### Comparison Table

| Concept            | Focus                      | When to Use                       |
| ------------------ | -------------------------- | --------------------------------- |
| **DDD**            | Modeling business domain   | Complex business logic            |
| **TDD**            | Test-first development     | Always (quality-focused projects) |
| **BDD**            | Behavior specifications    | Collaboration with stakeholders   |
| **ATDD**           | Acceptance criteria        | Requirements clarity needed       |
| **EDA**            | Event-based communication  | Loose coupling, scalability       |
| **Event Sourcing** | State as event history     | Audit trail, time travel needed   |
| **CQRS**           | Separate read/write models | Different read/write requirements |
| **Hexagonal**      | Isolate business logic     | Technology independence           |

---

### Combining These Concepts

These concepts often work together:

```typescript
// DDD + TDD + Hexagonal Architecture + Event Sourcing

// 1. Write test first (TDD)
describe('OrderService', () => {
  it('should place order and emit event', async () => {
    const eventStore = new InMemoryEventStore();
    const orderService = new OrderService(eventStore);

    const order = await orderService.placeOrder(customerId, items);

    expect(order.status).toBe('CONFIRMED');

    const events = eventStore.getEvents(order.orderId);
    expect(events).toContainEqual(
      expect.objectContaining({ type: 'OrderPlacedEvent' }),
    );
  });
});

// 2. Implement with DDD concepts
class Order {
  // Entity (DDD)
  constructor(
    public readonly orderId: string, // Identity
    private items: OrderItem[], // Value Objects
    private status: OrderStatus,
  ) {}

  placeOrder(): DomainEvent[] {
    // Domain Logic
    this.validateOrder();
    this.status = OrderStatus.PLACED;
    return [new OrderPlacedEvent(this.orderId)]; // Domain Event
  }

  private validateOrder(): void {
    if (this.items.length === 0) {
      throw new OrderValidationError('Order must have items');
    }
  }
}

// 3. Use Hexagonal Architecture
interface OrderRepository {
  // Port
  save(order: Order): Promise<void>;
}

class PostgresOrderRepository implements OrderRepository {
  // Adapter
  async save(order: Order): Promise<void> {
    // Implementation
  }
}

// 4. Event Sourcing
class OrderService {
  constructor(
    private eventStore: EventStore,
    private orderRepository: OrderRepository,
  ) {}

  async placeOrder(customerId: string, items: OrderItem[]): Promise<Order> {
    const order = new Order(generateId(), items, OrderStatus.PENDING);
    const events = order.placeOrder();

    // Store events (Event Sourcing)
    await this.eventStore.appendEvents(order.orderId, events);

    // Update read model (CQRS)
    await this.orderRepository.save(order);

    return order;
  }
}
```

---

## Summary

**Key Takeaways:**

1. **DDD:** Focus on business domain, use ubiquitous language, model the domain accurately
2. **TDD:** Write tests first, drive design through tests, refactor confidently
3. **BDD:** Specify behavior in business terms, collaborate with stakeholders
4. **EDA:** Communicate through events, decouple components
5. **Event Sourcing:** Store events, not just state, complete audit trail
6. **CQRS:** Separate reads and writes for optimization
7. **Hexagonal:** Isolate business logic, make it technology-independent

These are tools in your toolbox - use them when they solve real problems, not just because they're trendy!
