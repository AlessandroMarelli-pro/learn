# Node.js

## What is Node.js?

**Node.js** is an open-source, cross-platform JavaScript runtime environment that executes JavaScript code outside of a web browser. Built on Chrome's V8 JavaScript engine, Node.js enables developers to use JavaScript for server-side scripting, allowing the creation of dynamic web applications with a unified language across both client and server.

### Key Characteristics

- **Asynchronous and Event-Driven**: All APIs are non-blocking, meaning the server doesn't wait for data to return before moving to the next task
- **Single-Threaded**: Uses a single-threaded model with event looping, making it highly scalable for I/O-heavy operations
- **Fast Execution**: Built on V8 engine, which compiles JavaScript directly to native machine code
- **NPM Ecosystem**: Access to the world's largest package registry with over a million packages
- **Cross-Platform**: Runs on Windows, macOS, Linux, and various Unix systems

### Important topics summary

## Event Loop

### Simple Explanation

Think of the event loop as a waiter in a restaurant. Instead of taking one order, going to the kitchen, waiting for the food, and then taking the next order, the waiter takes multiple orders and checks back when each dish is ready.

The event loop constantly checks: "Is there any completed task I need to handle?" It picks up finished operations (like database queries or file reads) and executes their callback functions. It never stops - it keeps looping and checking for work.

### Technical Explanation

The event loop is a C++ implementation (in libuv library) that manages the execution of asynchronous operations. It operates in distinct phases:

1. **Timers phase:** Executes callbacks scheduled by `setTimeout()` and `setInterval()`
2. **Pending callbacks phase:** Executes I/O callbacks deferred to the next loop iteration
3. **Idle, prepare phase:** Internal use only
4. **Poll phase:** Retrieves new I/O events, executes I/O callbacks (except close callbacks, timers, and setImmediate)
5. **Check phase:** Executes `setImmediate()` callbacks
6. **Close callbacks phase:** Executes close event callbacks (e.g., `socket.on('close')`)

The event loop uses a **callback queue** (task queue) and **microtask queue**. Microtasks (Promises, `process.nextTick()`) have higher priority and execute between phases. `process.nextTick()` has the highest priority and executes before microtasks.

**Flow:** Call Stack → Web APIs/Node APIs → Callback Queue → Event Loop → Call Stack

When the call stack is empty, the event loop pushes the next callback from the queue onto the stack for execution.

### Why It's Useful

It allows Node.js to handle thousands of operations without getting stuck waiting. While one operation is processing (like reading a file), Node.js can start working on other tasks.

---

## Asynchronous

### Simple Explanation

Asynchronous means "not happening at the same time" or "don't wait for me."

When you make an asynchronous call, your program says: "Start this task, but don't make me wait for it to finish. I'll continue doing other things, and you tell me when you're done."

**Example:** When you order food delivery, you don't sit by the door waiting. You continue watching TV, and the delivery person calls when they arrive. That's asynchronous.

**Synchronous** would be standing at the door doing nothing until the food arrives.

### Technical Explanation

Asynchronous operations in Node.js are handled through several mechanisms:

#### 1. Callback Pattern

```javascript
fs.readFile('file.txt', (err, data) => {
  // executed when operation completes
});
// continues immediately without waiting
```

#### 2. Promises

```javascript
const promise = fs.promises.readFile('file.txt');
promise.then((data) => {}).catch((err) => {});
```

#### 3. Async/Await (syntactic sugar over Promises)

```javascript
async function read() {
  const data = await fs.promises.readFile('file.txt');
}
```

Under the hood, Node.js uses **libuv** (a C library) to handle asynchronous I/O. For file operations and DNS lookups, libuv maintains a **thread pool** (default 4 threads, configurable via `UV_THREADPOOL_SIZE`). For network I/O, it uses OS-level asynchronous mechanisms (epoll on Linux, kqueue on macOS, IOCP on Windows).

The JavaScript execution is non-blocking - the main thread delegates I/O operations to libuv, which either uses the thread pool or OS mechanisms, then notifies the event loop when complete via callbacks.

---

## Event-Driven

### Simple Explanation

Event-driven means your program reacts to events (things that happen) rather than following a strict step-by-step sequence.

Your code says: "When X happens, do Y." You register listeners for events like "user clicked a button," "file finished reading," or "data arrived from database."

**Example:** It's like setting multiple alarms on your phone. Each alarm triggers a different action when it goes off. Your day isn't one rigid sequence - you respond to events as they happen.

### Technical Explanation

Node.js implements the **Observer pattern** through the EventEmitter class. Objects emit named events that trigger registered listener functions.

```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter {}

const emitter = new MyEmitter();

// Register listener
emitter.on('event', (arg) => {
  console.log('Event occurred:', arg);
});

// Emit event
emitter.emit('event', 'some data');
```

#### Key Technical Aspects

- Listeners are stored in an array per event name
- Listeners execute **synchronously** in the order they were registered
- `once()` registers a listener that auto-removes after first execution
- `removeListener()` or `off()` unregisters listeners
- EventEmitter is the base class for many Node.js core modules (streams, http.Server, net.Socket)

#### Memory Management

Be careful with listeners - they can cause memory leaks if not removed. Node.js warns when more than 10 listeners are registered for a single event (configurable via `setMaxListeners()`).

---

## Single-Threaded & Why It's Scalable for I/O

### Simple Explanation

A thread is like a worker. Single-threaded means Node.js has only one main worker executing your JavaScript code at a time. No parallel execution of your JS code.

#### Why This Makes I/O Operations Scalable

This seems counterintuitive, but here's the magic:

Most I/O operations (database queries, file reading, network requests) involve **waiting**, not actual CPU work. The hard work happens outside Node.js - in the database server, file system, or network card.

**Traditional multi-threaded approach:** Create a new thread (worker) for each request. Each thread waits for its I/O operation. Threads consume memory (about 1-2MB each). With 10,000 concurrent users, you need 10,000 threads = 10-20GB of memory just for threads!

**Node.js single-threaded approach:** One thread handles all requests. When it encounters I/O, it says "start this operation and notify me when done" then immediately moves to the next request. No waiting, no blocking. That one thread can juggle thousands of I/O operations because it's never actually waiting - it's delegating the waiting to the operating system.

#### Simple Analogy

- **Multi-threaded:** 100 cashiers, each serving one customer and waiting while they count their money
- **Single-threaded Node.js:** 1 super-fast cashier who takes everyone's order, sends them to count money on the side, and calls them back when ready

### Technical Explanation

#### Architecture

- **Main thread:** Executes JavaScript code, runs the event loop
- **libuv thread pool:** Handles file I/O, DNS lookups, crypto operations (4 threads by default)
- **V8 background threads:** Garbage collection, compilation

#### Why I/O Scalable

##### 1. Non-blocking I/O

Traditional blocking I/O:

```
read(fd) -> thread blocked until data available -> return data
Context switches, thread overhead
```

Node.js non-blocking I/O:

```
read(fd, callback) -> returns immediately
OS notifies when ready -> callback queued -> event loop executes callback
```

##### 2. Memory Efficiency

- Thread stack: ~1-2MB per thread
- 10,000 threads = 10-20GB just for stacks
- Node.js: Single thread + small callback overhead
- Can handle 10,000+ concurrent connections with ~100MB

##### 3. No Context Switching Overhead

- Thread context switch: 1-100 microseconds
- Thousands of threads = significant CPU time wasted on scheduling
- Single thread: Zero context switching for JS execution

##### 4. C10K Problem Solution

Node.js solves the classic C10K problem (handling 10,000 concurrent connections) by:

- Using system calls like `epoll` (Linux), `kqueue` (BSD/macOS), `IOCP` (Windows)
- These OS mechanisms efficiently monitor thousands of file descriptors
- O(1) complexity for event notification vs O(n) for traditional `select()`

#### Technical Tradeoff

```javascript
// BAD - blocks the event loop
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
app.get('/fib', (req, res) => {
  res.send(fibonacci(40)); // Blocks entire server!
});

// GOOD - offload CPU work
const { Worker } = require('worker_threads');
app.get('/fib', (req, res) => {
  const worker = new Worker('./fib-worker.js');
  worker.postMessage(40);
  worker.on('message', (result) => res.send(result));
});
```

#### Concurrency Model

- **Concurrency:** Multiple operations in progress (Node.js excels)
- **Parallelism:** Multiple operations executing simultaneously (requires worker threads or child processes)

Node.js achieves high concurrency without parallelism for I/O because waiting is not computing. The OS handles actual I/O in parallel while Node.js coordinates efficiently.

#### Performance Metrics

- Traditional thread-per-request: ~10,000 concurrent connections max
- Node.js event-driven: 100,000+ concurrent connections possible
- Limitation: CPU-bound tasks block the single thread

### Important Note

This works brilliantly for I/O-heavy operations (APIs, web servers, real-time apps). For CPU-intensive tasks (complex calculations, image processing), single-threaded can be a limitation since that one thread gets busy computing instead of coordinating.

This is why Node.js dominates for: REST APIs, WebSocket servers, streaming applications, microservices - all I/O-heavy with minimal CPU processing per request.

### Use Cases

Node.js excels at:

- RESTful APIs and microservices
- Real-time applications (chat, collaboration tools, gaming)
- Streaming applications
- Single-page applications (SPAs)
- Command-line tools
- I/O-bound applications

**Not ideal for**: CPU-intensive operations that would block the single-threaded event loop.

---

## Table of Contents

### I. Overview

- **[(a) Introduction](<./I%20-%20Overview/(a)%20Introduction.md>)** - Getting started with Node.js fundamentals
- **[(b) Usage](<./I%20-%20Overview/(b)%20usage.md>)** - Common use cases and best practices
- **[(c) Security](<./I%20-%20Overview/(c)%20Security.md>)** - Security considerations and best practices

### II. HTTP

- **[HTTP](./II%20-%20HTTP/http.md)** - Building HTTP servers and handling requests

### III. Files

- **[(a) Files](<./III%20-%20Files%20/(a)%20files.md>)** - File operations and manipulation
- **[(b) File Systems](<./III%20-%20Files%20/(b)%20filesystems.md>)** - Working with the file system API

### IV. Asynchronous Programming

- **[(a) Callbacks & Flow](<./IV%20-%20Asynchronous/(a)%20Callbacks%20&%20flow.md>)** - Understanding callbacks and control flow
- **[(b) Promises](<./IV%20-%20Asynchronous/(b)%20Promises.md>)** - Promise-based asynchronous patterns
- **[(c) Timers](<./IV%20-%20Asynchronous/(c)%20Timers.md>)** - setTimeout, setInterval, and timer management
- **[(d) Event Loop](<./IV%20-%20Asynchronous/(d)%20Event%20Loop.md>)** - Deep dive into the event loop mechanism
- **[(e) setImmediate](<./IV%20-%20Asynchronous/(e)%20setImmediate.md>)** - Understanding setImmediate and its use cases

### V. TypeScript

- **[Overview](./V%20-%20Typescript/overview.md)** - Using TypeScript with Node.js

### VI. Modules

- **[(a) Streams](<./VI%20-%20Modules/(a)%20Streams.md>)** - Working with readable, writable, and transform streams
- **[(b) Publish](<./VI%20-%20Modules/(b)%20Publish.md>)** - Publishing and managing npm packages

### VII. Diagnostics

- **[(a) Memory](<./VII%20-%20Diagnostics/(a)%20memory.md>)** - Memory management and debugging
- **[(b) Performances](<./VII%20-%20Diagnostics/(b)%20performances.md>)** - Performance optimization and profiling

---

## Learning Path

For beginners, we recommend following this learning sequence:

1. **Start with Overview** (Section I) - Understand what Node.js is and how to use it
2. **Learn HTTP basics** (Section II) - Build your first server
3. **Master Asynchronous Programming** (Section IV) - Critical for effective Node.js development
4. **Explore Files and Streams** (Sections III & VI) - Handle data efficiently
5. **Optimize and Debug** (Section VII) - Write production-ready code
6. **Add TypeScript** (Section V) - Enhance code quality and maintainability

---

## Resources

- [Official Node.js Documentation](https://nodejs.org/docs/latest/api/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [NPM Registry](https://www.npmjs.com/)

```

```
