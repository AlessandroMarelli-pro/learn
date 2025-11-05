# Node.js - Intermediate Level Answers

---

## 1. Explain the difference between `process.nextTick()` and `setImmediate()`

**process.nextTick():**
- Executes before event loop continues
- Before any I/O operations
- Highest priority (can starve I/O)

**setImmediate():**
- Executes in check phase of event loop
- After I/O operations
- Designed for deferring execution

**Order:** nextTick → Promises → setImmediate

**Use nextTick:** Emit events, handle errors immediately
**Use setImmediate:** Break up long operations, defer work

---

## 2. How would you implement rate limiting in a Node.js API?

**Approaches:**

**1. In-memory (single server):**
- Store request counts per IP
- Reset after time window
- Simple but doesn't scale

**2. Redis-based (distributed):**
- Shared state across servers
- Atomic increment operations
- Sliding window or fixed window

**3. Token bucket algorithm:**
- Tokens refill at fixed rate
- Request consumes token
- Smooth rate limiting

**4. express-rate-limit package:**
- Ready-made solution
- Configurable windows and limits
- Multiple store options

---

## 3. What are the different types of streams and when would you use each?

**1. Readable streams:**
- Read data from source
- Examples: file reads, HTTP requests
- Use for: Processing large files, API responses

**2. Writable streams:**
- Write data to destination
- Examples: file writes, HTTP responses
- Use for: Saving data, sending responses

**3. Duplex streams:**
- Both readable and writable
- Examples: TCP sockets, WebSocket
- Use for: Two-way communication

**4. Transform streams:**
- Modify data while reading/writing
- Examples: compression, encryption
- Use for: Data processing pipelines

**Piping:** Connect streams together for efficient data flow.

---

## 4. How do you handle uncaught exceptions and unhandled promise rejections?

**Uncaught exceptions:**
- `process.on('uncaughtException')`
- Log error and exit gracefully
- Should not continue running

**Unhandled rejections:**
- `process.on('unhandledRejection')`
- Log error and exit
- Node.js will crash by default in future versions

**Graceful shutdown:**
- Close server (stop accepting new requests)
- Finish pending requests
- Close database connections
- Exit process

**Prevention:**
- Always use try-catch with async/await
- Add .catch() to all Promises
- Use error handling middleware in Express

---

## 5. Explain the Worker Threads API and when to use it

**Worker Threads:** True parallel execution for CPU-intensive tasks.

**When to use:**
- Heavy computation (image processing, data parsing)
- CPU-bound operations
- Operations blocking event loop

**When NOT to use:**
- I/O operations (already async)
- Simple tasks (overhead not worth it)

**Features:**
- Separate V8 instances
- Share memory with SharedArrayBuffer
- Message passing
- Thread pools

**Alternative:** Child processes for isolation, cluster for scaling.

---

## 6. How would you implement authentication and authorization in a Node.js API?

**Authentication (Who are you?):**

**Approaches:**
- **JWT:** Stateless, scalable, good for APIs
- **Session:** Server-side state, traditional
- **OAuth:** Third-party authentication

**Authorization (What can you do?):**

**Patterns:**
- **Role-based (RBAC):** User has roles with permissions
- **Attribute-based (ABAC):** Rules based on attributes
- **Resource-based:** Ownership checks

**Implementation:**
- Middleware for authentication
- Guards/decorators for authorization
- Store permissions in token or database
- Check permissions before operations

---

## 7. What are the best practices for error handling in Express.js?

**Strategies:**

**1. Async error wrapper:**
- Catch async errors automatically
- Pass to error handler

**2. Error handling middleware:**
- 4 parameters: `(err, req, res, next)`
- Must be last middleware
- Centralized error processing

**3. Operational vs programmer errors:**
- Operational: Expected (validation, not found)
- Programmer: Bugs (null reference, syntax)
- Handle operational, crash on programmer errors

**4. Error classes:**
- Custom error types
- Include status codes
- Add context

**5. Logging:**
- Log all errors
- Include request context
- Use error tracking service

---

## 8. How do you implement pagination in a REST API?

**Methods:**

**1. Offset-based (traditional):**
- Query params: `?page=2&limit=20`
- Skip and limit in database
- Simple but slow for large offsets

**2. Cursor-based (recommended):**
- Use unique identifier (ID, timestamp)
- More efficient for large datasets
- No missing/duplicate items

**3. Page-based with metadata:**
- Return total count, total pages
- Include pagination links
- Good for UI pagination

**Response format:**
- Include data array
- Pagination metadata (page, total, hasNext)
- Links to next/previous pages

---

## 9. Explain how to use Node.js child processes

**Child process types:**

**1. spawn:**
- Stream-based
- Large data output
- Long-running processes

**2. exec:**
- Buffered output
- Shell commands
- Small output

**3. execFile:**
- Like exec but no shell
- More secure
- Faster

**4. fork:**
- Special case of spawn
- Node.js processes
- IPC communication

**Use cases:**
- Run scripts in other languages
- Execute system commands
- Parallel processing
- Isolate risky operations

---

## 10. What is the difference between `spawn`, `exec`, `execFile`, and `fork`?

**spawn:**
- Streams data
- No shell by default
- Best for large output or long processes

**exec:**
- Buffers output (size limit)
- Uses shell
- Convenient for shell commands

**execFile:**
- Like exec but without shell
- More secure
- Direct binary execution

**fork:**
- Spawn Node.js process
- Built-in IPC channel
- Share objects between processes

**Choosing:** spawn for streams, exec for convenience, execFile for security, fork for Node.js.

---

## 11. How would you implement logging in a production Node.js application?

**Requirements:**
- Multiple log levels (error, warn, info, debug)
- Structured logging (JSON)
- Different transports (file, console, service)
- Performance (async, non-blocking)
- Context (request IDs, user info)

**Libraries:**
- **Winston:** Popular, flexible, many transports
- **Bunyan:** JSON-first, structured
- **Pino:** Fastest, low overhead

**Best practices:**
- Don't log sensitive data
- Include correlation IDs
- Aggregate logs centrally
- Set appropriate levels per environment
- Sample high-volume logs

---

## 12. Explain how to handle file uploads in Node.js

**Libraries:**
- **Multer:** Express middleware, most popular
- **Formidable:** Low-level, more control
- **Busboy:** Streaming parser

**Considerations:**
- File size limits
- File type validation
- Virus scanning
- Temporary storage
- Final destination (local, S3, etc.)

**Security:**
- Validate file types (not just extension)
- Limit file size
- Sanitize filenames
- Store outside web root
- Use random filenames

---

## 13. What are the security best practices for Node.js applications?

**Essential practices:**

**1. Dependencies:**
- Regular updates
- Audit with `npm audit`
- Use lock files

**2. Secrets:**
- Never commit to git
- Use environment variables
- Rotate regularly

**3. Helmet middleware:**
- Security headers
- XSS protection
- Content Security Policy

**4. Input validation:**
- Sanitize all input
- Parameterized queries
- Escape output

**5. Rate limiting:**
- Prevent abuse
- API throttling

**6. HTTPS:**
- Encrypt in transit
- Redirect HTTP

**7. Authentication:**
- Secure password storage (bcrypt)
- Session security
- JWT best practices

---

## 14. How do you implement caching strategies in Node.js?

**Caching layers:**

**1. In-memory (Node.js):**
- Simple object or Map
- Fast but lost on restart
- Single server only

**2. Redis:**
- Distributed cache
- Persistent
- Pub/sub support

**3. HTTP caching:**
- Cache-Control headers
- ETags
- CDN integration

**Strategies:**
- **Cache-aside:** App checks cache, loads on miss
- **Write-through:** Write to cache and DB simultaneously
- **Write-behind:** Write to cache, async to DB
- **Refresh-ahead:** Preload before expiration

**Invalidation:** Time-based (TTL), event-based, manual

---

## 15. Explain how to profile and debug Node.js applications

**Profiling tools:**

**1. Built-in:**
- `--inspect` flag (Chrome DevTools)
- `--prof` (V8 profiler)
- Performance hooks API

**2. External:**
- **Clinic.js:** Visual profiling
- **0x:** Flamegraphs
- **autocannon:** Load testing

**Debugging:**
- Chrome DevTools
- VS Code debugger
- Breakpoints and stepping
- Memory heap snapshots

**Monitoring:**
- CPU usage
- Memory consumption
- Event loop lag
- Active handles

**Production:** Use APM tools (New Relic, Datadog, Dynatrace)
