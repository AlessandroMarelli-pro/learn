I'll create a comprehensive Design Patterns interview preparation guide with both simple and technical explanations.

# Design Patterns - Interview Preparation

## What Are Design Patterns?

### Simple Explanation

Design patterns are proven solutions to common problems that developers face repeatedly. They're like recipes in cooking - tested blueprints that help you solve specific problems efficiently.

Instead of reinventing the wheel every time you face a problem like "how do I ensure only one instance of a class exists?" or "how do I add functionality without modifying existing code?", you can use an established pattern that others have refined over time.

### Technical Explanation

Design patterns are reusable solutions to recurring problems in software design. They represent best practices evolved over time by experienced software developers. Originally catalogued by the "Gang of Four" (GoF) in their 1994 book "Design Patterns: Elements of Reusable Object-Oriented Software."

**Key characteristics:**

- **Language-agnostic:** Concepts apply across programming languages
- **Not finished code:** Templates for solving problems, not copy-paste solutions
- **Proven solutions:** Battle-tested in real-world scenarios
- **Communication tool:** Shared vocabulary among developers

**Three main categories:**

1. **Creational Patterns:** Object creation mechanisms
2. **Structural Patterns:** Object composition and relationships
3. **Behavioral Patterns:** Communication between objects

---

## Creational Patterns

### Singleton Pattern

#### Simple Explanation

Singleton ensures that a class has only one instance throughout your application and provides a global point of access to it.

**Example:** Think of a database connection pool. You don't want multiple pools competing - you want one pool that everyone uses. Or a logging system - one logger that all parts of your app write to.

**When to use:**

- Configuration manager (one set of app settings)
- Database connection pool (share connections)
- Logger (centralized logging)
- Cache manager

#### Technical Explanation

**Intent:** Ensure a class has only one instance and provide a global access point to it.

**Implementation (TypeScript):**

```typescript
class Singleton {
  private static instance: Singleton;
  private constructor() {
    // Private constructor prevents instantiation from outside
    console.log('Singleton instance created');
  }

  public static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }

  public someMethod(): void {
    console.log('Singleton method called');
  }
}

// Usage
const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2); // true - same instance
```

**Thread-safe implementation (Java):**

```java
public class Singleton {
    // Eager initialization (thread-safe)
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// Or lazy with double-checked locking
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Modern JavaScript/TypeScript approach:**

```typescript
// Using module scope (automatically singleton)
class ConfigManager {
  private config: Record<string, any> = {};

  loadConfig(config: Record<string, any>): void {
    this.config = { ...this.config, ...config };
  }

  get(key: string): any {
    return this.config[key];
  }
}

export const configManager = new ConfigManager();

// Usage in other files
import { configManager } from './config';
configManager.loadConfig({ apiUrl: 'https://api.example.com' });
```

**Advantages:**

- Controlled access to sole instance
- Reduced namespace pollution
- Permits refinement of operations and representation
- Lazy initialization possible

**Disadvantages:**

- Difficult to unit test (global state)
- Violates Single Responsibility Principle (manages its own lifecycle)
- Can hide dependencies
- Thread-safety concerns in multi-threaded environments
- Considered an anti-pattern by some (global state)

**Real-world examples:**

- `Logger` instances
- Database connection pools
- Application configuration
- Hardware interface access (printer spooler)

---

### Factory Pattern

#### Simple Explanation

Factory pattern provides a way to create objects without specifying the exact class of object that will be created. It's like ordering from a menu - you ask for "pizza" without knowing exactly how the kitchen makes it.

**Example:** You have different types of notifications (email, SMS, push). Instead of deciding which notification class to create everywhere in your code, you ask a factory: "give me a notification sender" and it decides which one based on user preferences.

#### Technical Explanation

**Intent:** Define an interface for creating objects, but let subclasses decide which class to instantiate.

**Types:**

1. **Simple Factory:** Not a GoF pattern, but commonly used
2. **Factory Method:** Defines interface, subclasses implement
3. **Abstract Factory:** Families of related objects

**Simple Factory (TypeScript):**

```typescript
// Product interface
interface Notification {
  send(message: string): void;
}

// Concrete products
class EmailNotification implements Notification {
  send(message: string): void {
    console.log(`Sending email: ${message}`);
  }
}

class SMSNotification implements Notification {
  send(message: string): void {
    console.log(`Sending SMS: ${message}`);
  }
}

class PushNotification implements Notification {
  send(message: string): void {
    console.log(`Sending push notification: ${message}`);
  }
}

// Simple Factory
class NotificationFactory {
  static createNotification(type: string): Notification {
    switch (type) {
      case 'email':
        return new EmailNotification();
      case 'sms':
        return new SMSNotification();
      case 'push':
        return new PushNotification();
      default:
        throw new Error(`Unknown notification type: ${type}`);
    }
  }
}

// Usage
const notification = NotificationFactory.createNotification('email');
notification.send('Hello World!');
```

**Factory Method Pattern:**

```typescript
// Product interface
interface Transport {
  deliver(): void;
}

// Concrete products
class Truck implements Transport {
  deliver(): void {
    console.log('Delivering by land in a truck');
  }
}

class Ship implements Transport {
  deliver(): void {
    console.log('Delivering by sea in a ship');
  }
}

// Creator (abstract)
abstract class Logistics {
  abstract createTransport(): Transport;

  planDelivery(): void {
    const transport = this.createTransport();
    transport.deliver();
  }
}

// Concrete creators
class RoadLogistics extends Logistics {
  createTransport(): Transport {
    return new Truck();
  }
}

class SeaLogistics extends Logistics {
  createTransport(): Transport {
    return new Ship();
  }
}

// Usage
const roadLogistics = new RoadLogistics();
roadLogistics.planDelivery(); // Delivers by truck

const seaLogistics = new SeaLogistics();
seaLogistics.planDelivery(); // Delivers by ship
```

**Abstract Factory Pattern:**

```typescript
// Abstract products
interface Button {
  render(): void;
}

interface Checkbox {
  render(): void;
}

// Concrete products - Windows
class WindowsButton implements Button {
  render(): void {
    console.log('Rendering Windows button');
  }
}

class WindowsCheckbox implements Checkbox {
  render(): void {
    console.log('Rendering Windows checkbox');
  }
}

// Concrete products - Mac
class MacButton implements Button {
  render(): void {
    console.log('Rendering Mac button');
  }
}

class MacCheckbox implements Checkbox {
  render(): void {
    console.log('Rendering Mac checkbox');
  }
}

// Abstract factory
interface GUIFactory {
  createButton(): Button;
  createCheckbox(): Checkbox;
}

// Concrete factories
class WindowsFactory implements GUIFactory {
  createButton(): Button {
    return new WindowsButton();
  }

  createCheckbox(): Checkbox {
    return new WindowsCheckbox();
  }
}

class MacFactory implements GUIFactory {
  createButton(): Button {
    return new MacButton();
  }

  createCheckbox(): Checkbox {
    return new MacCheckbox();
  }
}

// Client code
class Application {
  private button: Button;
  private checkbox: Checkbox;

  constructor(factory: GUIFactory) {
    this.button = factory.createButton();
    this.checkbox = factory.createCheckbox();
  }

  render(): void {
    this.button.render();
    this.checkbox.render();
  }
}

// Usage
const os = 'Windows'; // or 'Mac'
const factory = os === 'Windows' ? new WindowsFactory() : new MacFactory();
const app = new Application(factory);
app.render();
```

**Advantages:**

- Loose coupling between creator and concrete products
- Single Responsibility Principle (creation code in one place)
- Open/Closed Principle (can add new products without changing existing code)
- Encapsulates object creation logic

**Disadvantages:**

- Code can become more complicated (many new subclasses)
- Requires creating a factory for each product type

**When to use:**

- When you don't know exact types/dependencies beforehand
- When you want to provide a library/framework extension point
- When you want to save system resources by reusing existing objects
- When object creation is complex

---

### Builder Pattern

#### Simple Explanation

Builder pattern constructs complex objects step by step. Instead of having a constructor with 10 parameters (confusing!), you build the object piece by piece with clear, readable code.

**Example:** Building a custom burger - you don't pass 15 arguments to a constructor. Instead: `burger.addCheese().addLettuce().addTomato().build()`. Much clearer!

#### Technical Explanation

**Intent:** Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

**Implementation (TypeScript):**

```typescript
// Product
class House {
  walls?: string;
  doors?: number;
  windows?: number;
  roof?: string;
  garage?: boolean;
  pool?: boolean;

  describe(): void {
    console.log(
      `House with ${this.walls} walls, ${this.doors} doors, ${this.windows} windows`,
    );
    if (this.roof) console.log(`Roof: ${this.roof}`);
    if (this.garage) console.log('Has garage');
    if (this.pool) console.log('Has pool');
  }
}

// Builder
class HouseBuilder {
  private house: House;

  constructor() {
    this.house = new House();
  }

  setWalls(material: string): HouseBuilder {
    this.house.walls = material;
    return this; // Return this for method chaining
  }

  setDoors(count: number): HouseBuilder {
    this.house.doors = count;
    return this;
  }

  setWindows(count: number): HouseBuilder {
    this.house.windows = count;
    return this;
  }

  setRoof(type: string): HouseBuilder {
    this.house.roof = type;
    return this;
  }

  addGarage(): HouseBuilder {
    this.house.garage = true;
    return this;
  }

  addPool(): HouseBuilder {
    this.house.pool = true;
    return this;
  }

  build(): House {
    return this.house;
  }
}

// Usage
const house = new HouseBuilder()
  .setWalls('brick')
  .setDoors(2)
  .setWindows(6)
  .setRoof('tile')
  .addGarage()
  .addPool()
  .build();

house.describe();
```

**With Director (optional orchestration):**

```typescript
// Director - knows how to build specific configurations
class HouseDirector {
  private builder: HouseBuilder;

  constructor(builder: HouseBuilder) {
    this.builder = builder;
  }

  buildMinimalHouse(): House {
    return this.builder
      .setWalls('wood')
      .setDoors(1)
      .setWindows(2)
      .setRoof('shingles')
      .build();
  }

  buildLuxuryHouse(): House {
    return this.builder
      .setWalls('stone')
      .setDoors(3)
      .setWindows(12)
      .setRoof('slate')
      .addGarage()
      .addPool()
      .build();
  }
}

// Usage
const builder = new HouseBuilder();
const director = new HouseDirector(builder);

const minimalHouse = director.buildMinimalHouse();
const luxuryHouse = director.buildLuxuryHouse();
```

**Real-world example - HTTP Request Builder:**

```typescript
class HttpRequest {
  url: string;
  method: string;
  headers: Record<string, string>;
  body?: any;
  timeout?: number;

  constructor(builder: HttpRequestBuilder) {
    this.url = builder.url;
    this.method = builder.method;
    this.headers = builder.headers;
    this.body = builder.body;
    this.timeout = builder.timeout;
  }

  async send(): Promise<Response> {
    return fetch(this.url, {
      method: this.method,
      headers: this.headers,
      body: this.body ? JSON.stringify(this.body) : undefined,
    });
  }
}

class HttpRequestBuilder {
  url: string = '';
  method: string = 'GET';
  headers: Record<string, string> = {};
  body?: any;
  timeout?: number;

  setUrl(url: string): HttpRequestBuilder {
    this.url = url;
    return this;
  }

  setMethod(method: string): HttpRequestBuilder {
    this.method = method;
    return this;
  }

  addHeader(key: string, value: string): HttpRequestBuilder {
    this.headers[key] = value;
    return this;
  }

  setBody(body: any): HttpRequestBuilder {
    this.body = body;
    return this;
  }

  setTimeout(timeout: number): HttpRequestBuilder {
    this.timeout = timeout;
    return this;
  }

  build(): HttpRequest {
    if (!this.url) throw new Error('URL is required');
    return new HttpRequest(this);
  }
}

// Usage
const request = new HttpRequestBuilder()
  .setUrl('https://api.example.com/users')
  .setMethod('POST')
  .addHeader('Content-Type', 'application/json')
  .addHeader('Authorization', 'Bearer token123')
  .setBody({ name: 'John', email: 'john@example.com' })
  .setTimeout(5000)
  .build();

request.send();
```

**Advantages:**

- Constructs objects step-by-step
- Reuses construction code for different representations
- Single Responsibility Principle (complex construction isolated)
- Immutable objects (build once)
- More readable than telescoping constructors

**Disadvantages:**

- Increases overall code complexity (many new classes)
- Requires creating a builder for each product

**When to use:**

- Objects with many optional parameters
- Complex object creation requiring multiple steps
- When you want immutable objects
- When the same construction process should create different representations

---

### Prototype Pattern

#### Simple Explanation

Prototype pattern creates new objects by copying existing objects (prototypes) instead of creating from scratch. It's like using "copy-paste" instead of recreating a document from scratch.

**Example:** You have a fully configured game character. Instead of recreating all settings for similar characters, you clone the prototype and modify what's different.

#### Technical Explanation

**Intent:** Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

**Implementation (TypeScript):**

```typescript
// Prototype interface
interface Prototype {
  clone(): Prototype;
}

// Concrete prototype
class Enemy implements Prototype {
  name: string;
  health: number;
  damage: number;
  position: { x: number; y: number };
  equipment: string[];

  constructor(
    name: string,
    health: number,
    damage: number,
    position: { x: number; y: number },
  ) {
    this.name = name;
    this.health = health;
    this.damage = damage;
    this.position = position;
    this.equipment = [];
  }

  // Shallow clone
  clone(): Enemy {
    const cloned = new Enemy(
      this.name,
      this.health,
      this.damage,
      { ...this.position }, // Clone position object
    );
    cloned.equipment = [...this.equipment]; // Clone array
    return cloned;
  }

  // Deep clone (if nested objects are complex)
  deepClone(): Enemy {
    return JSON.parse(JSON.stringify(this)) as Enemy;
  }

  describe(): void {
    console.log(
      `${this.name} - HP: ${this.health}, DMG: ${this.damage}, Pos: (${this.position.x}, ${this.position.y})`,
    );
  }
}

// Usage
const orcPrototype = new Enemy('Orc', 100, 20, { x: 0, y: 0 });
orcPrototype.equipment.push('axe', 'shield');

// Create multiple orcs by cloning
const orc1 = orcPrototype.clone();
orc1.position = { x: 10, y: 20 };

const orc2 = orcPrototype.clone();
orc2.position = { x: 30, y: 40 };
orc2.health = 150; // Make this one stronger

orc1.describe();
orc2.describe();
```

**Prototype Registry (Prototype Manager):**

```typescript
class ShapeRegistry {
  private shapes: Map<string, Prototype> = new Map();

  registerShape(key: string, shape: Prototype): void {
    this.shapes.set(key, shape);
  }

  createShape(key: string): Prototype | undefined {
    const prototype = this.shapes.get(key);
    return prototype ? prototype.clone() : undefined;
  }
}

// Abstract shape
abstract class Shape implements Prototype {
  abstract clone(): Shape;
  abstract draw(): void;
}

// Concrete shapes
class Circle extends Shape {
  constructor(public radius: number, public color: string) {
    super();
  }

  clone(): Circle {
    return new Circle(this.radius, this.color);
  }

  draw(): void {
    console.log(`Drawing ${this.color} circle with radius ${this.radius}`);
  }
}

class Rectangle extends Shape {
  constructor(
    public width: number,
    public height: number,
    public color: string,
  ) {
    super();
  }

  clone(): Rectangle {
    return new Rectangle(this.width, this.height, this.color);
  }

  draw(): void {
    console.log(`Drawing ${this.color} rectangle ${this.width}x${this.height}`);
  }
}

// Usage
const registry = new ShapeRegistry();

// Register prototypes
registry.registerShape('redCircle', new Circle(10, 'red'));
registry.registerShape('blueRectangle', new Rectangle(20, 30, 'blue'));

// Create objects from prototypes
const circle1 = registry.createShape('redCircle');
const circle2 = registry.createShape('redCircle');
const rect1 = registry.createShape('blueRectangle');

circle1?.draw();
rect1?.draw();
```

**JavaScript Prototypal Inheritance:**

```javascript
// JavaScript uses prototypes natively
const carPrototype = {
  wheels: 4,
  start: function () {
    console.log('Starting engine...');
  },
};

// Create objects using prototype
const car1 = Object.create(carPrototype);
car1.color = 'red';

const car2 = Object.create(carPrototype);
car2.color = 'blue';

car1.start(); // Uses prototype method
console.log(car1.wheels); // 4 from prototype
```

**Advantages:**

- Reduces need for subclassing
- Hides complexity of creating objects
- Performance benefit (cloning vs. new instantiation)
- Dynamic addition/removal of objects at runtime
- Specifies new objects by varying values

**Disadvantages:**

- Cloning complex objects with circular references can be tricky
- Deep vs. shallow copy concerns
- Each subclass must implement clone()

**When to use:**

- Object creation is expensive (database calls, network requests)
- Similar objects that differ only in state
- Avoiding subclass explosion
- Dynamic object composition at runtime

---

## Structural Patterns

### Adapter Pattern

#### Simple Explanation

Adapter (or Wrapper) makes incompatible interfaces work together. It's like a power adapter for traveling - your US plug doesn't fit EU sockets, so you use an adapter.

**Example:** You have an old library that expects XML, but your app uses JSON. An adapter converts JSON to XML so the library can understand it.

#### Technical Explanation

**Intent:** Convert the interface of a class into another interface clients expect. Adapter lets classes work together that couldn't otherwise because of incompatible interfaces.

**Two types:**

1. **Class Adapter:** Uses inheritance (multiple inheritance)
2. **Object Adapter:** Uses composition (preferred in languages without multiple inheritance)

**Implementation (TypeScript):**

```typescript
// Target interface (what client expects)
interface MediaPlayer {
  play(filename: string): void;
}

// Adaptee (incompatible interface)
class AdvancedMediaPlayer {
  playVlc(filename: string): void {
    console.log(`Playing VLC file: ${filename}`);
  }

  playMp4(filename: string): void {
    console.log(`Playing MP4 file: ${filename}`);
  }
}

// Adapter
class MediaAdapter implements MediaPlayer {
  private advancedPlayer: AdvancedMediaPlayer;

  constructor(private audioType: string) {
    this.advancedPlayer = new AdvancedMediaPlayer();
  }

  play(filename: string): void {
    if (this.audioType === 'vlc') {
      this.advancedPlayer.playVlc(filename);
    } else if (this.audioType === 'mp4') {
      this.advancedPlayer.playMp4(filename);
    }
  }
}

// Client
class AudioPlayer implements MediaPlayer {
  play(filename: string): void {
    const extension = filename.split('.').pop()?.toLowerCase();

    if (extension === 'mp3') {
      console.log(`Playing MP3 file: ${filename}`);
    } else if (extension === 'vlc' || extension === 'mp4') {
      const adapter = new MediaAdapter(extension);
      adapter.play(filename);
    } else {
      console.log(`Invalid format: ${extension}`);
    }
  }
}

// Usage
const player = new AudioPlayer();
player.play('song.mp3');
player.play('movie.vlc');
player.play('video.mp4');
```

**Real-world example - API Adapter:**

```typescript
// Old payment system interface
interface OldPaymentSystem {
  makePayment(amount: number, accountNumber: string): boolean;
}

class LegacyPaymentSystem implements OldPaymentSystem {
  makePayment(amount: number, accountNumber: string): boolean {
    console.log(`Processing ${amount} to account ${accountNumber}`);
    return true;
  }
}

// New payment interface
interface ModernPaymentProcessor {
  processPayment(paymentDetails: {
    amount: number;
    currency: string;
    recipient: string;
  }): Promise<{ success: boolean; transactionId: string }>;
}

// Adapter
class PaymentAdapter implements ModernPaymentProcessor {
  constructor(private legacySystem: OldPaymentSystem) {}

  async processPayment(paymentDetails: {
    amount: number;
    currency: string;
    recipient: string;
  }): Promise<{ success: boolean; transactionId: string }> {
    // Convert new format to old format
    const success = this.legacySystem.makePayment(
      paymentDetails.amount,
      paymentDetails.recipient,
    );

    return {
      success,
      transactionId: `TXN-${Date.now()}`,
    };
  }
}

// Usage
const legacySystem = new LegacyPaymentSystem();
const modernProcessor = new PaymentAdapter(legacySystem);

modernProcessor
  .processPayment({
    amount: 100,
    currency: 'USD',
    recipient: 'ACC123456',
  })
  .then((result) => console.log('Payment result:', result));
```

**Two-way Adapter:**

```typescript
interface Celsius {
  getTemperature(): number;
}

interface Fahrenheit {
  getTemperature(): number;
}

class CelsiusSensor implements Celsius {
  constructor(private temp: number) {}

  getTemperature(): number {
    return this.temp;
  }
}

class TwoWayTemperatureAdapter implements Celsius, Fahrenheit {
  constructor(private celsius?: Celsius, private fahrenheit?: Fahrenheit) {}

  // Celsius interface
  getTemperature(): number {
    if (this.celsius) {
      return this.celsius.getTemperature();
    } else if (this.fahrenheit) {
      return ((this.fahrenheit.getTemperature() - 32) * 5) / 9;
    }
    return 0;
  }

  // Could add Fahrenheit conversion method
  getTemperatureFahrenheit(): number {
    return (this.getTemperature() * 9) / 5 + 32;
  }
}
```

**Advantages:**

- Single Responsibility Principle (separates interface conversion)
- Open/Closed Principle (introduce new adapters without changing existing code)
- Reuse existing code with incompatible interfaces
- Works with multiple adaptees

**Disadvantages:**

- Increases overall complexity (new interfaces and classes)
- Sometimes simpler to just change the service class

**When to use:**

- Want to use existing class with incompatible interface
- Need to create reusable class that cooperates with unrelated classes
- Need to use several existing subclasses without adapting each one
- Third-party library integration

---

### Decorator Pattern

#### Simple Explanation

Decorator adds new functionality to objects dynamically without modifying their structure. It's like adding toppings to a pizza - the base pizza stays the same, but you add extras on top.

**Example:** A simple text box. You can decorate it with: scroll bars, borders, or both. Each decorator adds functionality without changing the original text box class.

#### Technical Explanation

**Intent:** Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality.

**Implementation (TypeScript):**

```typescript
// Component interface
interface Coffee {
  cost(): number;
  description(): string;
}

// Concrete component
class SimpleCoffee implements Coffee {
  cost(): number {
    return 5;
  }

  description(): string {
    return 'Simple coffee';
  }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
  constructor(protected coffee: Coffee) {}

  abstract cost(): number;
  abstract description(): string;
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 1.5;
  }

  description(): string {
    return this.coffee.description() + ', milk';
  }
}

class SugarDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 0.5;
  }

  description(): string {
    return this.coffee.description() + ', sugar';
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost(): number {
    return this.coffee.cost() + 2;
  }

  description(): string {
    return this.coffee.description() + ', whipped cream';
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
console.log(`${coffee.description()} - $${coffee.cost()}`);
// "Simple coffee - $5"

coffee = new MilkDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// "Simple coffee, milk - $6.5"

coffee = new SugarDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// "Simple coffee, milk, sugar - $7"

coffee = new WhipDecorator(coffee);
console.log(`${coffee.description()} - $${coffee.cost()}`);
// "Simple coffee, milk, sugar, whipped cream - $9"
```

**Real-world example - Data Stream:**

```typescript
interface DataSource {
  writeData(data: string): void;
  readData(): string;
}

// Concrete component
class FileDataSource implements DataSource {
  private data: string = '';

  constructor(private filename: string) {}

  writeData(data: string): void {
    console.log(`Writing to file ${this.filename}: ${data}`);
    this.data = data;
  }

  readData(): string {
    console.log(`Reading from file ${this.filename}`);
    return this.data;
  }
}

// Base decorator
abstract class DataSourceDecorator implements DataSource {
  constructor(protected wrappee: DataSource) {}

  abstract writeData(data: string): void;
  abstract readData(): string;
}

// Encryption decorator
class EncryptionDecorator extends DataSourceDecorator {
  writeData(data: string): void {
    const encrypted = this.encrypt(data);
    this.wrappee.writeData(encrypted);
  }

  readData(): string {
    const data = this.wrappee.readData();
    return this.decrypt(data);
  }

  private encrypt(data: string): string {
    // Simple encryption (in reality, use proper encryption)
    return Buffer.from(data).toString('base64');
  }

  private decrypt(data: string): string {
    return Buffer.from(data, 'base64').toString('utf-8');
  }
}

// Compression decorator
class CompressionDecorator extends DataSourceDecorator {
  writeData(data: string): void {
    const compressed = this.compress(data);
    this.wrappee.writeData(compressed);
  }

  readData(): string {
    const data = this.wrappee.readData();
    return this.decompress(data);
  }

  private compress(data: string): string {
    console.log('Compressing data...');
    return `COMPRESSED(${data})`;
  }

  private decompress(data: string): string {
    console.log('Decompressing data...');
    return data.replace('COMPRESSED(', '').replace(')', '');
  }
}

// Usage
let source: DataSource = new FileDataSource('data.txt');

// Add encryption
source = new EncryptionDecorator(source);

// Add compression
source = new CompressionDecorator(source);

// Write and read - automatically compressed and encrypted
source.writeData('Hello World!');
const data = source.readData();
console.log('Final data:', data);
```

**TypeScript Decorator Syntax (Experimental):**

```typescript
// Method decorator
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(5, 3);
// Logs: "Calling add with args: [5, 3]"
// Logs: "Result: 8"
```

**Advantages:**

- More flexible than static inheritance
- Responsibilities can be added/removed at runtime
- Combine several decorators for complex functionality
- Single Responsibility Principle (divide functionality among classes)
- Open/Closed Principle (extend without modifying)

**Disadvantages:**

- Can be difficult to remove specific wrapper from stack
- Hard to implement decorator order-independent
- Initial configuration code might look ugly
- Many small objects in design

**When to use:**

- Add responsibilities to objects dynamically
- Responsibilities can be withdrawn
- Extension by subclassing is impractical
- Need multiple optional features combinations

---

### Facade Pattern

#### Simple Explanation

Facade provides a simplified interface to a complex system. It's like a restaurant menu - you don't need to know how the kitchen works, you just order from a simple menu.

**Example:** Starting a computer involves many steps (load BIOS, initialize hardware, load OS). But you just press one power button - that's a facade hiding all the complexity.

#### Technical Explanation

**Intent:** Provide a unified interface to a set of interfaces in a subsystem. Facade defines a higher-level interface that makes the subsystem easier to use.

**Implementation (TypeScript):**

```typescript
// Complex subsystem classes
class CPU {
  freeze(): void {
    console.log('CPU: Freezing...');
  }

  jump(position: number): void {
    console.log(`CPU: Jumping to ${position}`);
  }

  execute(): void {
    console.log('CPU: Executing...');
  }
}

class Memory {
  load(position: number, data: string): void {
    console.log(`Memory: Loading data "${data}" at position ${position}`);
  }
}

class HardDrive {
  read(sector: number, size: number): string {
    console.log(`HardDrive: Reading ${size} bytes from sector ${sector}`);
    return 'bootloader_data';
  }
}

// Facade
class ComputerFacade {
  private cpu: CPU;
  private memory: Memory;
  private hardDrive: HardDrive;

  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }

  start(): void {
    console.log('ComputerFacade: Starting computer...');
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    console.log('ComputerFacade: Computer started!');
  }
}

// Usage - Simple interface
const computer = new ComputerFacade();
computer.start();
// User doesn't need to know about CPU, Memory, HardDrive
```

**Real-world example - Video Conversion:**

```typescript
// Complex subsystem
class VideoFile {
  constructor(public filename: string) {}
}

class OggCompressionCodec {
  compress(file: VideoFile): Buffer {
    console.log('Compressing to OGG format...');
    return Buffer.from('ogg_data');
  }
}

class MPEG4CompressionCodec {
  compress(file: VideoFile): Buffer {
    console.log('Compressing to MPEG4 format...');
    return Buffer.from('mpeg4_data');
  }
}

class CodecFactory {
  static extract(file: VideoFile): OggCompressionCodec | MPEG4CompressionCodec {
    const ext = file.filename.split('.').pop();
    if (ext === 'ogg') return new OggCompressionCodec();
    return new MPEG4CompressionCodec();
  }
}

class BitrateReader {
  static read(file: VideoFile, codec: any): Buffer {
    console.log('Reading bitrate...');
    return Buffer.from('bitrate_data');
  }

  static convert(buffer: Buffer, codec: any): Buffer {
    console.log('Converting bitrate...');
    return buffer;
  }
}

class AudioMixer {
  fix(result: Buffer): Buffer {
    console.log('Fixing audio...');
    return result;
  }
}

// Facade - Simplified interface
class VideoConverter {
  convert(filename: string, format: string): Buffer {
    console.log(
      `VideoConverter: Starting conversion of ${filename} to ${format}`,
    );

    const file = new VideoFile(filename);
    const sourceCodec = CodecFactory.extract(file);

    let destinationCodec: any;
    if (format === 'mp4') {
      destinationCodec = new MPEG4CompressionCodec();
    } else {
      destinationCodec = new OggCompressionCodec();
    }

    const buffer = BitrateReader.read(file, sourceCodec);
    let result = BitrateReader.convert(buffer, destinationCodec);
    result = new AudioMixer().fix(result);

    console.log('VideoConverter: Conversion complete!');
    return result;
  }
}

// Usage - Much simpler!
const converter = new VideoConverter();
converter.convert('video.ogg', 'mp4');
// User doesn't need to know about codecs, bitrates, audio mixing
```

**API Facade example:**

```typescript
// Complex third-party API
class TwitterAPI {
  getUser(username: string): any {
    console.log(`Twitter: Getting user ${username}`);
    return { id: 1, name: username, platform: 'twitter' };
  }

  getUserTweets(userId: number): any[] {
    console.log(`Twitter: Getting tweets for user ${userId}`);
    return [{ text: 'Tweet 1' }, { text: 'Tweet 2' }];
  }
}

class FacebookAPI {
  getUserProfile(email: string): any {
    console.log(`Facebook: Getting profile for ${email}`);
    return { id: 2, name: 'User', platform: 'facebook' };
  }

  getUserPosts(profileId: number): any[] {
    console.log(`Facebook: Getting posts for profile ${profileId}`);
    return [{ content: 'Post 1' }, { content: 'Post 2' }];
  }
}

// Facade - Unified interface
class SocialMediaFacade {
  private twitter: TwitterAPI;
  private facebook: FacebookAPI;

  constructor() {
    this.twitter = new TwitterAPI();
    this.facebook = new FacebookAPI();
  }

  getUserPosts(identifier: string, platform: 'twitter' | 'facebook'): string[] {
    if (platform === 'twitter') {
      const user = this.twitter.getUser(identifier);
      const tweets = this.twitter.getUserTweets(user.id);
      return tweets.map((t) => t.text);
    } else {
      const profile = this.facebook.getUserProfile(identifier);
      const posts = this.facebook.getUserPosts(profile.id);
      return posts.map((p) => p.content);
    }
  }
}

// Usage - Simple unified interface
const socialMedia = new SocialMediaFacade();
const twitterPosts = socialMedia.getUserPosts('john_doe', 'twitter');
const facebookPosts = socialMedia.getUserPosts('john@example.com', 'facebook');
```

**Advantages:**

- Isolates clients from subsystem complexity
- Loose coupling between subsystems and clients
- Layers the subsystem (organize complexity)
- Easier to use, understand, test

**Disadvantages:**

- Facade can become a god object coupled to all classes
- Might not provide enough customization for advanced users

**When to use:**

- Simple interface needed to complex subsystem
- Many dependencies between clients and implementation classes
- Want to layer subsystems
- Decouple subsystem from clients and other subsystems

**Facade vs Adapter:**

- Facade: Simplifies interface, works with multiple classes
- Adapter: Converts interface, typically works with single class

---

### Proxy Pattern

#### Simple Explanation

Proxy provides a surrogate or placeholder for another object to control access to it. It's like a bodyguard - you can't access the celebrity directly, you go through their bodyguard who controls access.

**Example:** A credit card is a proxy for a bank account. You don't carry your entire bank account, but the card gives you access to it.

#### Technical Explanation

**Intent:** Provide a surrogate or placeholder for another object to control access to it.

**Types of proxies:**

1. **Virtual Proxy:** Lazy initialization (create expensive objects on demand)
2. **Protection Proxy:** Access control (permissions)
3. **Remote Proxy:** Represents remote object (RPC, web services)
4. **Logging Proxy:** Logs requests
5. **Caching Proxy:** Caches results
6. **Smart Reference:** Additional actions when object accessed (reference counting, locking)

**Implementation (TypeScript):**

**1. Virtual Proxy (Lazy Initialization):**

```typescript
// Subject interface
interface Image {
  display(): void;
}

// Real subject
class RealImage implements Image {
  private filename: string;

  constructor(filename: string) {
    this.filename = filename;
    this.loadFromDisk();
  }

  private loadFromDisk(): void {
    console.log(`Loading image from disk: ${this.filename}`);
    // Expensive operation
  }

  display(): void {
    console.log(`Displaying image: ${this.filename}`);
  }
}

// Proxy
class ImageProxy implements Image {
  private realImage: RealImage | null = null;
  private filename: string;

  constructor(filename: string) {
    this.filename = filename;
  }

  display(): void {
    // Lazy initialization - create real object only when needed
    if (!this.realImage) {
      this.realImage = new RealImage(this.filename);
    }
    this.realImage.display();
  }
}

// Usage
const image1 = new ImageProxy('photo1.jpg');
const image2 = new ImageProxy('photo2.jpg');

console.log('Images created but not loaded yet');

image1.display(); // Now it loads and displays
// Output:
// Loading image from disk: photo1.jpg
// Displaying image: photo1.jpg

image1.display(); // Uses already loaded image
// Output:
// Displaying image: photo1.jpg
```

**2. Protection Proxy (Access Control):**

```typescript
interface BankAccount {
  deposit(amount: number): void;
  withdraw(amount: number): void;
  getBalance(): number;
}

class RealBankAccount implements BankAccount {
  private balance: number = 0;

  deposit(amount: number): void {
    this.balance += amount;
    console.log(`Deposited $${amount}. New balance: $${this.balance}`);
  }

  withdraw(amount: number): void {
    if (amount <= this.balance) {
      this.balance -= amount;
      console.log(`Withdrew $${amount}. New balance: $${this.balance}`);
    } else {
      console.log('Insufficient funds');
    }
  }

  getBalance(): number {
    return this.balance;
  }
}

class BankAccountProxy implements BankAccount {
  private realAccount: RealBankAccount;
  private owner: string;

  constructor(owner: string) {
    this.owner = owner;
    this.realAccount = new RealBankAccount();
  }

  deposit(amount: number): void {
    // Anyone can deposit
    this.realAccount.deposit(amount);
  }

  withdraw(amount: number): void {
    // Only owner can withdraw
    if (this.checkAccess()) {
      this.realAccount.withdraw(amount);
    } else {
      console.log('Access denied: You are not the account owner');
    }
  }

  getBalance(): number {
    if (this.checkAccess()) {
      return this.realAccount.getBalance();
    } else {
      console.log('Access denied: You are not the account owner');
      return 0;
    }
  }

  private checkAccess(): boolean {
    // In reality, check actual authentication
    console.log('Checking access rights...');
    return true; // Simplified
  }
}

// Usage
const account = new BankAccountProxy('John Doe');
account.deposit(1000);
account.withdraw(500);
console.log(`Balance: $${account.getBalance()}`);
```

**3. Caching Proxy:**

```typescript
interface DataService {
  fetchData(id: string): any;
}

class RealDataService implements DataService {
  fetchData(id: string): any {
    console.log(`Fetching data from database for ID: ${id}`);
    // Simulate expensive database operation
    return { id, data: `Data for ${id}`, timestamp: Date.now() };
  }
}

class CachingProxy implements DataService {
  private realService: RealDataService;
  private cache: Map<string, { data: any; timestamp: number }> = new Map();
  private cacheDuration: number = 5000; // 5 seconds

  constructor() {
    this.realService = new RealDataService();
  }

  fetchData(id: string): any {
    const cached = this.cache.get(id);
    const now = Date.now();

    if (cached && now - cached.timestamp < this.cacheDuration) {
      console.log(`Returning cached data for ID: ${id}`);
      return cached.data;
    }

    console.log(`Cache miss or expired for ID: ${id}`);
    const data = this.realService.fetchData(id);
    this.cache.set(id, { data, timestamp: now });
    return data;
  }
}

// Usage
const service = new CachingProxy();

service.fetchData('user-123'); // Fetches from database
service.fetchData('user-123'); // Returns from cache
service.fetchData('user-456'); // Fetches from database
service.fetchData('user-123'); // Returns from cache
```

**4. Logging Proxy:**

```typescript
interface Service {
  operation(data: string): string;
}

class RealService implements Service {
  operation(data: string): string {
    return `Processed: ${data}`;
  }
}

class LoggingProxy implements Service {
  private realService: Service;

  constructor(realService: Service) {
    this.realService = realService;
  }

  operation(data: string): string {
    console.log(`[LOG] Before operation - Input: ${data}`);
    const startTime = Date.now();

    const result = this.realService.operation(data);

    const endTime = Date.now();
    console.log(
      `[LOG] After operation - Output: ${result}, Duration: ${
        endTime - startTime
      }ms`,
    );

    return result;
  }
}

// Usage
const service = new LoggingProxy(new RealService());
service.operation('test data');
```

**5. Remote Proxy (Simplified):**

```typescript
interface RemoteService {
  getData(): Promise<any>;
}

// Represents object on remote server
class RemoteServiceProxy implements RemoteService {
  private serverUrl: string;

  constructor(serverUrl: string) {
    this.serverUrl = serverUrl;
  }

  async getData(): Promise<any> {
    console.log(`Making remote call to ${this.serverUrl}`);

    // Simulate HTTP request
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve({ message: 'Data from remote server' });
      }, 1000);
    });
  }
}

// Usage
const remoteService = new RemoteServiceProxy('https://api.example.com/data');
remoteService.getData().then((data) => console.log(data));
```

**Combining Multiple Proxy Behaviors:**

```typescript
class CompositeProxy implements Image {
  private image: RealImage | null = null;
  private filename: string;
  private cache: { [key: string]: any } = {};

  constructor(filename: string) {
    this.filename = filename;
  }

  display(): void {
    // Logging
    console.log(`[LOG] Displaying ${this.filename}`);

    // Caching
    if (this.cache[this.filename]) {
      console.log('[CACHE] Using cached image');
      return;
    }

    // Access control
    if (!this.checkPermissions()) {
      console.log('[ACCESS DENIED]');
      return;
    }

    // Lazy initialization
    if (!this.image) {
      this.image = new RealImage(this.filename);
    }

    this.image.display();
    this.cache[this.filename] = true;
  }

  private checkPermissions(): boolean {
    // Check user permissions
    return true;
  }
}
```

**Advantages:**

- Controls access to object
- Can manage object lifecycle (lazy initialization)
- Works even if service object isn't ready or available
- Open/Closed Principle (introduce new proxies without changing service)
- Can add functionality without modifying service

**Disadvantages:**

- Code becomes more complicated
- Response from service might be delayed
- Additional layer of abstraction

**When to use:**

- Lazy initialization (virtual proxy)
- Access control (protection proxy)
- Local execution of remote service (remote proxy)
- Logging requests (logging proxy)
- Caching results (caching proxy)
- Reference counting (smart reference)

**Proxy vs Decorator:**

- Proxy: Controls access, manages lifecycle
- Decorator: Adds responsibilities
- Proxy usually manages object itself
- Decorator is controlled by client

---

## Behavioral Patterns

### Observer Pattern

#### Simple Explanation

Observer defines a one-to-many dependency so when one object changes state, all its dependents are notified automatically. It's like subscribing to a YouTube channel - when they upload a video, all subscribers get notified.

**Example:** Newsletter subscription. When new content is published, all subscribers receive it automatically without the publisher needing to know who they are.

#### Technical Explanation

**Intent:** Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

**Also known as:** Publish-Subscribe, Event-Subscriber, Listener

**Implementation (TypeScript):**

```typescript
// Observer interface
interface Observer {
  update(subject: Subject): void;
}

// Subject interface
interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// Concrete Subject
class WeatherStation implements Subject {
  private observers: Observer[] = [];
  private temperature: number = 0;
  private humidity: number = 0;
  private pressure: number = 0;

  attach(observer: Observer): void {
    const isExist = this.observers.includes(observer);
    if (isExist) {
      console.log('Observer already attached');
      return;
    }
    this.observers.push(observer);
    console.log('Observer attached');
  }

  detach(observer: Observer): void {
    const observerIndex = this.observers.indexOf(observer);
    if (observerIndex === -1) {
      console.log('Observer not found');
      return;
    }
    this.observers.splice(observerIndex, 1);
    console.log('Observer detached');
  }

  notify(): void {
    console.log('Notifying all observers...');
    for (const observer of this.observers) {
      observer.update(this);
    }
  }

  setMeasurements(
    temperature: number,
    humidity: number,
    pressure: number,
  ): void {
    this.temperature = temperature;
    this.humidity = humidity;
    this.pressure = pressure;
    this.notify();
  }

  getTemperature(): number {
    return this.temperature;
  }

  getHumidity(): number {
    return this.humidity;
  }

  getPressure(): number {
    return this.pressure;
  }
}

// Concrete Observers
class CurrentConditionsDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      console.log(
        `Current conditions: ${subject.getTemperature()}°C, ${subject.getHumidity()}% humidity`,
      );
    }
  }
}

class StatisticsDisplay implements Observer {
  private temperatures: number[] = [];

  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      this.temperatures.push(subject.getTemperature());
      const avg =
        this.temperatures.reduce((a, b) => a + b, 0) / this.temperatures.length;
      console.log(`Avg temperature: ${avg.toFixed(1)}°C`);
    }
  }
}

class ForecastDisplay implements Observer {
  update(subject: Subject): void {
    if (subject instanceof WeatherStation) {
      const pressure = subject.getPressure();
      if (pressure > 1020) {
        console.log('Forecast: Improving weather!');
      } else if (pressure < 1010) {
        console.log('Forecast: Watch out for rain!');
      } else {
        console.log('Forecast: More of the same');
      }
    }
  }
}

// Usage
const weatherStation = new WeatherStation();

const currentDisplay = new CurrentConditionsDisplay();
const statisticsDisplay = new StatisticsDisplay();
const forecastDisplay = new ForecastDisplay();

weatherStation.attach(currentDisplay);
weatherStation.attach(statisticsDisplay);
weatherStation.attach(forecastDisplay);

weatherStation.setMeasurements(25, 65, 1013);
console.log('---');
weatherStation.setMeasurements(27, 70, 1015);
console.log('---');

weatherStation.detach(forecastDisplay);
weatherStation.setMeasurements(22, 90, 1008);
```

**Event-driven implementation (more modern):**

```typescript
type EventHandler<T = any> = (data: T) => void;

class EventEmitter {
  private events: Map<string, EventHandler[]> = new Map();

  on(event: string, handler: EventHandler): void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(handler);
  }

  off(event: string, handler: EventHandler): void {
    const handlers = this.events.get(event);
    if (handlers) {
      const index = handlers.indexOf(handler);
      if (index !== -1) {
        handlers.splice(index, 1);
      }
    }
  }

  emit(event: string, data?: any): void {
    const handlers = this.events.get(event);
    if (handlers) {
      handlers.forEach((handler) => handler(data));
    }
  }

  once(event: string, handler: EventHandler): void {
    const onceHandler: EventHandler = (data) => {
      handler(data);
      this.off(event, onceHandler);
    };
    this.on(event, onceHandler);
  }
}

// Usage
class UserService extends EventEmitter {
  createUser(name: string, email: string): void {
    console.log(`Creating user: ${name}`);

    // Emit event
    this.emit('userCreated', { name, email, timestamp: Date.now() });
  }
}

const userService = new UserService();

// Subscribe to events
userService.on('userCreated', (data) => {
  console.log('Send welcome email to:', data.email);
});

userService.on('userCreated', (data) => {
  console.log('Log user creation:', data.name);
});

userService.once('userCreated', () => {
  console.log('This runs only once');
});

userService.createUser('John Doe', 'john@example.com');
console.log('---');
userService.createUser('Jane Smith', 'jane@example.com');
```

**Real-world example - Stock Market:**

```typescript
interface StockObserver {
  update(stock: string, price: number): void;
}

class Stock {
  private symbol: string;
  private price: number;
  private observers: StockObserver[] = [];

  constructor(symbol: string, price: number) {
    this.symbol = symbol;
    this.price = price;
  }

  addObserver(observer: StockObserver): void {
    this.observers.push(observer);
  }

  removeObserver(observer: StockObserver): void {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }

  setPrice(price: number): void {
    this.price = price;
    this.notifyObservers();
  }

  private notifyObservers(): void {
    this.observers.forEach((observer) => {
      observer.update(this.symbol, this.price);
    });
  }
}

class StockTrader implements StockObserver {
  constructor(private name: string, private buyThreshold: number) {}

  update(stock: string, price: number): void {
    if (price < this.buyThreshold) {
      console.log(
        `${this.name}: BUY ${stock} at $${price} (below threshold $${this.buyThreshold})`,
      );
    }
  }
}

class StockBroker implements StockObserver {
  private portfolio: Map<string, number> = new Map();

  update(stock: string, price: number): void {
    this.portfolio.set(stock, price);
    console.log(`Broker: Updated ${stock} to $${price} in portfolio`);
  }
}

// Usage
const appleStock = new Stock('AAPL', 150);

const trader1 = new StockTrader('Trader 1', 145);
const trader2 = new StockTrader('Trader 2', 140);
const broker = new StockBroker();

appleStock.addObserver(trader1);
appleStock.addObserver(trader2);
appleStock.addObserver(broker);

appleStock.setPrice(148);
console.log('---');
appleStock.setPrice(142);
console.log('---');
appleStock.setPrice(138);
```

**Advantages:**

- Open/Closed Principle (introduce new subscribers without changing publisher)
- Establishes relationships at runtime
- Loose coupling between subject and observers
- Support for broadcast communication

**Disadvantages:**

- Subscribers notified in random order
- Memory leaks if observers not properly unsubscribed
- Can cause unexpected cascading updates
- No guarantee observers notified (if unsubscribed)

**When to use:**

- Change in one object requires changing others (unknown number)
- Object should notify other objects without assumptions about who they are
- Event handling systems
- Model-View architecture (MVC, MVVM)
- Real-time data feeds

---

### Strategy Pattern

#### Simple Explanation

Strategy defines a family of algorithms, encapsulates each one, and makes them interchangeable. It's like different payment methods - credit card, PayPal, crypto - all accomplish the same goal (payment) but work differently.

**Example:** Navigation app - you can choose car, bike, or walk. Each strategy calculates a different route, but the app doesn't care which one you use.

#### Technical Explanation

**Intent:** Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

**Also known as:** Policy

**Implementation (TypeScript):**

```typescript
// Strategy interface
interface PaymentStrategy {
  pay(amount: number): void;
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
  constructor(
    private cardNumber: string,
    private cvv: string,
    private expiryDate: string,
  ) {}

  pay(amount: number): void {
    console.log(
      `Paid $${amount} using Credit Card ending in ${this.cardNumber.slice(
        -4,
      )}`,
    );
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number): void {
    console.log(`Paid $${amount} using PayPal account: ${this.email}`);
  }
}

class CryptoPayment implements PaymentStrategy {
  constructor(private walletAddress: string) {}

  pay(amount: number): void {
    console.log(
      `Paid $${amount} using Crypto wallet: ${this.walletAddress.slice(
        0,
        10,
      )}...`,
    );
  }
}

// Context
class ShoppingCart {
  private items: { name: string; price: number }[] = [];
  private paymentStrategy?: PaymentStrategy;

  addItem(name: string, price: number): void {
    this.items.push({ name, price });
  }

  setPaymentStrategy(strategy: PaymentStrategy): void {
    this.paymentStrategy = strategy;
  }

  checkout(): void {
    if (!this.paymentStrategy) {
      console.log('Please select a payment method');
      return;
    }

    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    console.log(`Total: $${total}`);
    this.paymentStrategy.pay(total);
    this.items = []; // Clear cart
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem('Laptop', 1000);
cart.addItem('Mouse', 50);

// Pay with credit card
cart.setPaymentStrategy(
  new CreditCardPayment('1234567890123456', '123', '12/25'),
);
cart.checkout();

// Add more items
cart.addItem('Keyboard', 100);

// Pay with PayPal
cart.setPaymentStrategy(new PayPalPayment('user@example.com'));
cart.checkout();

// Pay with Crypto
cart.setPaymentStrategy(
  new CryptoPayment('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb'),
);
cart.checkout();
```

**Real-world example - Compression Algorithms:**

```typescript
interface CompressionStrategy {
  compress(data: string): string;
  decompress(data: string): string;
}

class ZipCompression implements CompressionStrategy {
  compress(data: string): string {
    console.log('Compressing using ZIP algorithm...');
    return `ZIP(${data})`;
  }

  decompress(data: string): string {
    console.log('Decompressing ZIP...');
    return data.replace('ZIP(', '').replace(')', '');
  }
}

class RARCompression implements CompressionStrategy {
  compress(data: string): string {
    console.log('Compressing using RAR algorithm...');
    return `RAR(${data})`;
  }

  decompress(data: string): string {
    console.log('Decompressing RAR...');
    return data.replace('RAR(', '').replace(')', '');
  }
}

class GzipCompression implements CompressionStrategy {
  compress(data: string): string {
    console.log('Compressing using GZIP algorithm...');
    return `GZIP(${data})`;
  }

  decompress(data: string): string {
    console.log('Decompressing GZIP...');
    return data.replace('GZIP(', '').replace(')', '');
  }
}

class FileCompressor {
  private strategy: CompressionStrategy;

  constructor(strategy: CompressionStrategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy: CompressionStrategy): void {
    this.strategy = strategy;
  }

  compressFile(data: string): string {
    return this.strategy.compress(data);
  }

  decompressFile(data: string): string {
    return this.strategy.decompress(data);
  }
}

// Usage
const compressor = new FileCompressor(new ZipCompression());

let compressed = compressor.compressFile('Important data');
console.log('Compressed:', compressed);

// Change strategy at runtime
compressor.setStrategy(new RARCompression());
compressed = compressor.compressFile('More important data');
console.log('Compressed:', compressed);

// Change again
compressor.setStrategy(new GzipCompression());
const decompressed = compressor.decompressFile('GZIP(Some compressed data)');
console.log('Decompressed:', decompressed);
```

**Sorting strategies example:**

```typescript
interface SortStrategy<T> {
  sort(data: T[]): T[];
}

class BubbleSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Sorting using Bubble Sort...');
    const arr = [...data];
    // Bubble sort implementation
    for (let i = 0; i < arr.length; i++) {
      for (let j = 0; j < arr.length - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    return arr;
  }
}

class QuickSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Sorting using Quick Sort...');
    if (data.length <= 1) return data;

    const pivot = data[0];
    const left = data.slice(1).filter((x) => x <= pivot);
    const right = data.slice(1).filter((x) => x > pivot);

    return [...this.sort(left), pivot, ...this.sort(right)];
  }
}

class MergeSort<T> implements SortStrategy<T> {
  sort(data: T[]): T[] {
    console.log('Sorting using Merge Sort...');
    if (data.length <= 1) return data;

    const mid = Math.floor(data.length / 2);
    const left = this.sort(data.slice(0, mid));
    const right = this.sort(data.slice(mid));

    return this.merge(left, right);
  }

  private merge(left: T[], right: T[]): T[] {
    const result: T[] = [];
    let i = 0,
      j = 0;

    while (i < left.length && j < right.length) {
      if (left[i] <= right[j]) {
        result.push(left[i++]);
      } else {
        result.push(right[j++]);
      }
    }

    return [...result, ...left.slice(i), ...right.slice(j)];
  }
}

class Sorter<T> {
  constructor(private strategy: SortStrategy<T>) {}

  setStrategy(strategy: SortStrategy<T>): void {
    this.strategy = strategy;
  }

  sort(data: T[]): T[] {
    return this.strategy.sort(data);
  }
}

// Usage
const data = [64, 34, 25, 12, 22, 11, 90];

// Small dataset - use Bubble Sort
const sorter = new Sorter(new BubbleSort<number>());
console.log('Result:', sorter.sort(data));

// Large dataset - use Quick Sort
sorter.setStrategy(new QuickSort<number>());
console.log('Result:', sorter.sort(data));

// Stability required - use Merge Sort
sorter.setStrategy(new MergeSort<number>());
console.log('Result:', sorter.sort(data));
```

**Functional approach (without classes):**

```typescript
type DiscountStrategy = (price: number) => number;

const noDiscount: DiscountStrategy = (price) => price;

const seasonalDiscount: DiscountStrategy = (price) => price * 0.9; // 10% off

const blackFridayDiscount: DiscountStrategy = (price) => price * 0.5; // 50% off

const loyaltyDiscount: DiscountStrategy = (price) => price * 0.85; // 15% off

class Product {
  constructor(private name: string, private price: number) {}

  calculatePrice(discountStrategy: DiscountStrategy): number {
    return discountStrategy(this.price);
  }
}

// Usage
const product = new Product('Laptop', 1000);

console.log('No discount:', product.calculatePrice(noDiscount));
console.log('Seasonal:', product.calculatePrice(seasonalDiscount));
console.log('Black Friday:', product.calculatePrice(blackFridayDiscount));
console.log('Loyalty:', product.calculatePrice(loyaltyDiscount));
```

**Advantages:**

- Family of algorithms interchangeable
- Eliminates conditional statements for algorithm selection
- Open/Closed Principle (introduce new strategies without changing context)
- Runtime algorithm switching
- Isolates algorithm implementation from code that uses it

**Disadvantages:**

- Clients must be aware of different strategies
- Increases number of objects
- Communication overhead between strategy and context
- Modern languages support anonymous functions (can replace simple strategies)

**When to use:**

- Many related classes differ only in behavior
- Need different variants of algorithm
- Algorithm uses data clients shouldn't know about
- Class has massive conditional statements switching between algorithm variants

**Strategy vs State:**

- Strategy: Client chooses strategy
- State: Context changes state automatically
- Strategy: Algorithms do similar things differently
- State: Different behavior per state

---

### Command Pattern

#### Simple Explanation

Command encapsulates a request as an object, allowing you to parameterize clients with different requests, queue requests, log requests, and support undo operations.

It's like a restaurant order - the waiter doesn't cook, they just take your order (command) to the kitchen. The order slip can be queued, logged, or cancelled.

**Example:** Text editor - each action (cut, copy, paste, undo) is a command object. Commands can be stored for undo/redo functionality.

#### Technical Explanation

**Intent:** Encapsulate a request as an object, thereby letting you parametrize clients with different requests, queue or log requests, and support undoable operations.

**Also known as:** Action, Transaction

**Implementation (TypeScript):**

```typescript
// Command interface
interface Command {
  execute(): void;
  undo(): void;
}

// Receiver
class Light {
  private isOn: boolean = false;

  turnOn(): void {
    this.isOn = true;
    console.log('Light is ON');
  }

  turnOff(): void {
    this.isOn = false;
    console.log('Light is OFF');
  }

  getState(): string {
    return this.isOn ? 'ON' : 'OFF';
  }
}

// Concrete Commands
class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOn();
  }

  undo(): void {
    this.light.turnOff();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute(): void {
    this.light.turnOff();
  }

  undo(): void {
    this.light.turnOn();
  }
}

// Invoker
class RemoteControl {
  private history: Command[] = [];

  executeCommand(command: Command): void {
    command.execute();
    this.history.push(command);
  }

  undo(): void {
    const command = this.history.pop();
    if (command) {
      command.undo();
    } else {
      console.log('Nothing to undo');
    }
  }
}

// Usage
const light = new Light();
const lightOn = new LightOnCommand(light);
const lightOff = new LightOffCommand(light);

const remote = new RemoteControl();

remote.executeCommand(lightOn); // Light is ON
remote.executeCommand(lightOff); // Light is OFF
remote.undo(); // Light is ON (undo off)
remote.undo(); // Light is OFF (undo on)
remote.undo(); // Nothing to undo
```

**Real-world example - Text Editor:**

```typescript
interface EditorCommand {
  execute(): void;
  undo(): void;
}

class TextEditor {
  private text: string = '';

  getText(): string {
    return this.text;
  }

  setText(text: string): void {
    this.text = text;
  }

  insertText(position: number, newText: string): void {
    this.text =
      this.text.slice(0, position) + newText + this.text.slice(position);
    console.log(`Text: "${this.text}"`);
  }

  deleteText(position: number, length: number): void {
    this.text =
      this.text.slice(0, position) + this.text.slice(position + length);
    console.log(`Text: "${this.text}"`);
  }
}

class InsertCommand implements EditorCommand {
  constructor(
    private editor: TextEditor,
    private position: number,
    private text: string,
  ) {}

  execute(): void {
    this.editor.insertText(this.position, this.text);
  }

  undo(): void {
    this.editor.deleteText(this.position, this.text.length);
  }
}

class DeleteCommand implements EditorCommand {
  private deletedText: string = '';

  constructor(
    private editor: TextEditor,
    private position: number,
    private length: number,
  ) {}

  execute(): void {
    const text = this.editor.getText();
    this.deletedText = text.slice(this.position, this.position + this.length);
    this.editor.deleteText(this.position, this.length);
  }

  undo(): void {
    this.editor.insertText(this.position, this.deletedText);
  }
}

class CommandManager {
  private history: EditorCommand[] = [];
  private currentIndex: number = -1;

  execute(command: EditorCommand): void {
    // Remove any commands after current position (when new command executed after undo)
    this.history = this.history.slice(0, this.currentIndex + 1);

    command.execute();
    this.history.push(command);
    this.currentIndex++;
  }

  undo(): void {
    if (this.currentIndex >= 0) {
      this.history[this.currentIndex].undo();
      this.currentIndex--;
    } else {
      console.log('Nothing to undo');
    }
  }

  redo(): void {
    if (this.currentIndex < this.history.length - 1) {
      this.currentIndex++;
      this.history[this.currentIndex].execute();
    } else {
      console.log('Nothing to redo');
    }
  }
}

// Usage
const editor = new TextEditor();
const manager = new CommandManager();

manager.execute(new InsertCommand(editor, 0, 'Hello'));
manager.execute(new InsertCommand(editor, 5, ' World'));
manager.execute(new DeleteCommand(editor, 0, 6));

console.log('---Undo operations---');
manager.undo(); // Restores "Hello"
manager.undo(); // Removes " World"

console.log('---Redo operations---');
manager.redo(); // Re-adds " World"
manager.redo(); // Re-deletes "Hello"
```

**Macro Commands (Composite):**

```typescript
class MacroCommand implements Command {
  private commands: Command[] = [];

  add(command: Command): void {
    this.commands.push(command);
  }

  execute(): void {
    console.log('Executing macro command...');
    for (const command of this.commands) {
      command.execute();
    }
  }

  undo(): void {
    console.log('Undoing macro command...');
    // Undo in reverse order
    for (let i = this.commands.length - 1; i >= 0; i--) {
      this.commands[i].undo();
    }
  }
}

// Usage
const light1 = new Light();
const light2 = new Light();
const light3 = new Light();

const allLightsOn = new MacroCommand();
allLightsOn.add(new LightOnCommand(light1));
allLightsOn.add(new LightOnCommand(light2));
allLightsOn.add(new LightOnCommand(light3));

const remote2 = new RemoteControl();
remote2.executeCommand(allLightsOn); // All lights ON
remote2.undo(); // All lights OFF
```

**Queuing commands:**

```typescript
class CommandQueue {
  private queue: Command[] = [];

  enqueue(command: Command): void {
    this.queue.push(command);
    console.log(`Command enqueued. Queue size: ${this.queue.length}`);
  }

  processAll(): void {
    console.log('Processing all commands in queue...');
    while (this.queue.length > 0) {
      const command = this.queue.shift()!;
      command.execute();
    }
  }
}

// Usage - Batch processing
const queue = new CommandQueue();
queue.enqueue(new LightOnCommand(new Light()));
queue.enqueue(new LightOffCommand(new Light()));
queue.enqueue(new LightOnCommand(new Light()));

queue.processAll();
```

**Advantages:**

- Decouples sender and receiver
- Single Responsibility Principle (invoker vs. receiver)
- Open/Closed Principle (introduce new commands without changing existing)
- Implement undo/redo
- Implement deferred execution (queuing)
- Assemble set of simple commands into complex one

**Disadvantages:**

- Code becomes more complicated (many new classes)
- Indirection between sender and receiver

**When to use:**

- Parametrize objects with operations
- Queue operations, schedule execution, or execute remotely
- Implement reversible operations (undo/redo)
- Log changes for crash recovery
- Structure system around high-level operations built on primitive operations

---

This comprehensive guide covers the most important design patterns with both simple explanations and technical implementations. Good luck with your interview!

```

```

```

```
