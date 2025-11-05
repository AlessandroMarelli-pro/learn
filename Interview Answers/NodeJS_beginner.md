# Node.js - Beginner Level Answers

---

## 1. What is Node.js and how does it differ from browser JavaScript?

**Node.js:** JavaScript runtime built on Chrome's V8 engine for server-side execution.

**Key differences:**
- **Environment:** Server vs browser
- **Global object:** `global` vs `window`
- **APIs:** File system, HTTP, OS vs DOM, BOM
- **Module system:** CommonJS (`require`) vs ES modules/script tags
- **Execution:** Long-running process vs per-page
- **Access:** Full system access vs sandboxed

---

## 2. Explain the Node.js event-driven architecture

**Event-driven:** Operations emit events that trigger registered callbacks instead of blocking.

**Components:**
- **Event emitter:** Core pattern for pub-sub
- **Event loop:** Orchestrates execution
- **Callbacks:** Functions executed when events occur
- **Non-blocking I/O:** Operations don't wait for completion

**Benefits:**
- Handle many concurrent connections
- Efficient for I/O-heavy tasks
- Scalable with limited resources
- Single-threaded but concurrent

---

## 3. What is `package.json` and what are its key properties?

**package.json:** Project manifest file containing metadata, dependencies, and configuration.

**Key properties:**
- **name, version:** Package identification
- **main:** Entry point file
- **scripts:** Custom npm commands
- **dependencies:** Production packages
- **devDependencies:** Development-only packages
- **engines:** Required Node.js version
- **author, license:** Metadata

**Created with:** `npm init`

---

## 4. What is the difference between `require()` and ES6 `import`?

**`require()` (CommonJS):**
- Synchronous loading
- Dynamic (runtime)
- Can be conditional
- Returns any value
- Node.js default (traditionally)

**`import` (ES6 Modules):**
- Asynchronous loading
- Static (compile-time)
- Must be top-level
- Only exports
- Modern standard

**Node.js:** Supports both with `.mjs` extension or `"type": "module"` in package.json

---

## 5. What are streams in Node.js?

**Streams:** Objects for reading/writing data in chunks instead of loading everything into memory.

**Four types:**
1. **Readable:** Read data (files, HTTP requests)
2. **Writable:** Write data (files, HTTP responses)
3. **Duplex:** Both readable and writable (TCP sockets)
4. **Transform:** Modify data while reading/writing (compression)

**Benefits:**
- Memory efficient (process large files)
- Time efficient (start processing before all data arrives)
- Composable with pipes

---

## 6. Explain the purpose of `process.env`

**process.env:** Object containing environment variables.

**Use cases:**
- Configuration (database URLs, API keys)
- Environment detection (development, production)
- Port numbers
- Feature flags
- Secrets management

**Best practices:**
- Never commit secrets to version control
- Use `.env` files for local development
- Validate required variables on startup
- Use different values per environment

---

## 7. What is middleware in Express.js?

**Middleware:** Functions that execute during request-response cycle with access to `req`, `res`, and `next`.

**Types:**
- **Application-level:** `app.use()`
- **Router-level:** `router.use()`
- **Built-in:** `express.json()`, `express.static()`
- **Third-party:** `cors`, `helmet`, `morgan`
- **Error-handling:** 4 parameters including `err`

**Flow:** Request → Middleware 1 → Middleware 2 → ... → Route handler → Response

**Purpose:** Authentication, logging, parsing, error handling, etc.

---

## 8. How do you handle environment variables in Node.js applications?

**Methods:**

**1. dotenv package:**
- Load variables from `.env` file
- Popular and simple
- Good for development

**2. config module:**
- Hierarchical configurations
- Environment-specific files
- Supports JSON, YAML, etc.

**3. Direct `process.env`:**
- Access variables set by system/container
- Production deployment

**Best practice:** Use `.env` locally, environment variables in production, never commit secrets.

---

## 9. What is `npm` and what is `npx`?

**npm (Node Package Manager):**
- Install and manage packages
- Run scripts from package.json
- Publish packages to registry
- Comes with Node.js

**npx (Node Package Execute):**
- Execute packages without global install
- Always uses latest version (or specified)
- Runs binaries from local node_modules
- Temporary execution

**When to use:**
- **npm:** Install dependencies, run scripts
- **npx:** One-off commands, project scaffolding

---

## 10. Explain the purpose of the `cluster` module

**Cluster module:** Creates child processes (workers) to utilize multiple CPU cores.

**How it works:**
- Master process forks workers
- Workers share server port
- Load balanced across workers
- Master manages worker lifecycle

**Benefits:**
- Utilize all CPU cores
- Increased throughput
- Zero-downtime restarts
- Fault tolerance (restart dead workers)

**Use case:** Scale single-threaded Node.js to multi-core systems.

---

## 11. What are the differences between development and production dependencies?

**dependencies:**
- Required in production
- Installed with `npm install`
- Needed for app to run
- Examples: express, mongoose, lodash

**devDependencies:**
- Only needed during development
- Skipped with `npm install --production`
- Not deployed to production
- Examples: jest, nodemon, eslint, webpack

**Install:**
- Production: `npm install package`
- Development: `npm install -D package`

---

## 12. How do you read and write files in Node.js?

**Three approaches:**

**1. Synchronous (blocking):**
- Simple, sequential
- Blocks event loop
- Use only for startup scripts

**2. Asynchronous (callback):**
- Non-blocking
- Traditional Node.js style
- Callback hell risk

**3. Promise-based:**
- Modern approach
- Use with async/await
- Cleanest syntax

**Methods:** `fs.readFile`, `fs.writeFile`, `fs.appendFile`, `fs.unlink`, etc.

---

## 13. What is CORS and how do you handle it in Node.js?

**CORS (Cross-Origin Resource Sharing):** Security mechanism that controls which domains can access your API.

**Browser behavior:** Blocks requests from different origins unless server allows it.

**Solution in Node.js:**
- Use `cors` middleware package
- Configure allowed origins
- Set credentials, methods, headers

**Simple:** Allow all origins (development only)
**Strict:** Whitelist specific domains (production)

---

## 14. Explain what callback hell is and how to avoid it

**Callback hell (Pyramid of Doom):** Deeply nested callbacks making code hard to read and maintain.

**Causes:**
- Multiple async operations in sequence
- Each depending on previous result
- Error handling at each level

**Solutions:**
1. **Named functions:** Extract callbacks
2. **Promises:** Chain with `.then()`
3. **Async/await:** Write async code synchronously
4. **Async libraries:** async.js, Promise utilities

---

## 15. What is the purpose of `process.nextTick()`?

**process.nextTick():** Schedules callback to execute after current operation completes but before event loop continues.

**Execution order:**
1. Current operation finishes
2. `process.nextTick()` callbacks
3. Microtasks (Promises)
4. Event loop continues to next phase

**Use cases:**
- Emit events after constructor returns
- Ensure truly async execution
- Allow time for event listeners to be registered

**Warning:** Can starve event loop if used recursively.
