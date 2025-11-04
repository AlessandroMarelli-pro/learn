# Node.js

## What is Node.js?

**Node.js** is an open-source, cross-platform JavaScript runtime environment that executes JavaScript code outside of a web browser. Built on Chrome's V8 JavaScript engine, Node.js enables developers to use JavaScript for server-side scripting, allowing the creation of dynamic web applications with a unified language across both client and server.

### Key Characteristics

- **Asynchronous and Event-Driven**: All APIs are non-blocking, meaning the server doesn't wait for data to return before moving to the next task
- **Single-Threaded**: Uses a single-threaded model with event looping, making it highly scalable for I/O-heavy operations
- **Fast Execution**: Built on V8 engine, which compiles JavaScript directly to native machine code
- **NPM Ecosystem**: Access to the world's largest package registry with over a million packages
- **Cross-Platform**: Runs on Windows, macOS, Linux, and various Unix systems

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
- **[(a) Introduction](./I%20-%20Overview/(a)%20Introduction.md)** - Getting started with Node.js fundamentals
- **[(b) Usage](./I%20-%20Overview/(b)%20usage.md)** - Common use cases and best practices
- **[(c) Security](./I%20-%20Overview/(c)%20Security.md)** - Security considerations and best practices

### II. HTTP
- **[HTTP](./II%20-%20HTTP/http.md)** - Building HTTP servers and handling requests

### III. Files
- **[(a) Files](./III%20-%20Files%20/(a)%20files.md)** - File operations and manipulation
- **[(b) File Systems](./III%20-%20Files%20/(b)%20filesystems.md)** - Working with the file system API

### IV. Asynchronous Programming
- **[(a) Callbacks & Flow](./IV%20-%20Asynchronous/(a)%20Callbacks%20&%20flow.md)** - Understanding callbacks and control flow
- **[(b) Promises](./IV%20-%20Asynchronous/(b)%20Promises.md)** - Promise-based asynchronous patterns
- **[(c) Timers](./IV%20-%20Asynchronous/(c)%20Timers.md)** - setTimeout, setInterval, and timer management
- **[(d) Event Loop](./IV%20-%20Asynchronous/(d)%20Event%20Loop.md)** - Deep dive into the event loop mechanism
- **[(e) setImmediate](./IV%20-%20Asynchronous/(e)%20setImmediate.md)** - Understanding setImmediate and its use cases

### V. TypeScript
- **[Overview](./V%20-%20Typescript/overview.md)** - Using TypeScript with Node.js

### VI. Modules
- **[(a) Streams](./VI%20-%20Modules/(a)%20Streams.md)** - Working with readable, writable, and transform streams
- **[(b) Publish](./VI%20-%20Modules/(b)%20Publish.md)** - Publishing and managing npm packages

### VII. Diagnostics
- **[(a) Memory](./VII%20-%20Diagnostics/(a)%20memory.md)** - Memory management and debugging
- **[(b) Performances](./VII%20-%20Diagnostics/(b)%20performances.md)** - Performance optimization and profiling

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
