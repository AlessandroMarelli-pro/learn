# Domain-Driven Design (DDD) - Interview Preparation

## What is Domain-Driven Design?

### Simple Explanation

Domain-Driven Design (DDD) is an approach to software development that focuses on understanding and modeling the business domain (the real-world problem you're solving) as the foundation of your software design.

Instead of starting with databases or technical architecture, you start by deeply understanding the business, its rules, and its language. Then you build software that reflects that understanding.

**Example:** If you're building banking software, you don't just think about "saving data to a database." You think about "accounts," "transactions," "overdraft rules," "interest calculations" - the actual banking concepts that domain experts (bankers) understand.

It's like learning the language of the business first, then speaking that language in your code.

### Technical Explanation

DDD is a software design philosophy introduced by Eric Evans in his 2003 book "Domain-Driven Design: Tackling Complexity in the Heart of Software." It emphasizes collaboration between technical experts and domain experts to create a software model that accurately reflects business reality.

**Core principles:**

1. **Focus on the domain and domain logic**
2. **Base complex designs on a model of the domain**
3. **Collaborate with domain experts to improve the model iteratively**
4. **Use ubiquitous language for shared understanding**
5. **Bounded contexts to manage complexity**

**Key benefits:**

- Better alignment between business and software
- More maintainable code
- Handles complex business logic effectively
- Improves communication between teams
- Supports continuous evolution

**When to use DDD:**

- Complex business domains
- Long-term projects requiring maintainability
- Core business logic is complex and valuable
- Domain experts are available for collaboration

**When NOT to use DDD:**

- Simple CRUD applications
- Technical/infrastructure problems
- Short-term projects
- When business logic is trivial

---

## Ubiquitous Language

### Simple Explanation

Ubiquitous Language is a common vocabulary shared by developers, domain experts, business people, and everyone involved in the project. Everyone uses the same words to mean the same things.

**Bad example:**

- Developer says: "We'll update the user record in the database"
- Business person says: "We need to process the customer's order"
- They're talking about different things using different words!

**Good example (Ubiquitous Language):**

- Everyone says: "When we place an order, we reserve inventory and create a shipment"
- Same terms, same meaning, in conversations AND in code

### Technical Explanation

Ubiquitous Language is a structured language around the domain model that is used by all team members to connect all activities with the software. It's ubiquitous in the sense that it's used everywhere - in conversations, documentation, and most importantly, in the code itself.

**Key characteristics:**

- **Shared vocabulary:** Same terms used by business and tech
- **Reflected in code:** Class names, method names match business terms
- **Evolves over time:** Refined through continuous collaboration
- **Context-specific:** Can differ between bounded contexts

**Example - E-commerce:**

```typescript
// BAD - Technical language
class UserData {
  items: any[];

  processPayment() {
    // Process payment
  }

  updateInventory() {
    // Update stock
  }
}

// GOOD - Ubiquitous Language
class Order {
  orderItems: OrderItem[];

  placeOrder(): void {
    this.reserveInventory();
    this.processPayment();
    this.createShipment();
  }

  private reserveInventory(): void {
    // Reserve inventory for order items
  }

  private processPayment(): void {
    // Process payment
  }

  private createShipment(): void {
    // Create shipment
  }
}

class OrderItem {
  constructor(
    public product: Product,
    public quantity: number,
    public priceAtTimeOfOrder: Money,
  ) {}
}
```

**Benefits:**

- Reduces misunderstandings
- Makes code self-documenting
- Easier onboarding for new team members
- Bridges communication gap between business and tech
- Reveals missing concepts or ambiguities

**How to develop it:**

- Regular conversations with domain experts
- Document terms in glossary
- Refactor code when language evolves
- Challenge ambiguous terms
- Use domain terms
