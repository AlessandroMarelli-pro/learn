# Node.js Streams

## Overview

Streams provide a mechanism for processing data incrementally in chunks rather than loading entire datasets into memory. They can be thought of as "arrays over time" - enabling efficient handling of data that arrives sequentially or is too large to fit in memory.

Streams inherit from the `EventEmitter` class, making them event-driven and fully integrated with Node.js's asynchronous architecture.

---

## Why Use Streams?

### Three Core Benefits

1. **Memory Efficiency**: Process data in manageable chunks without loading complete datasets into memory
2. **Improved Response Time**: Begin processing immediately as data arrives, rather than waiting for complete payload delivery
3. **Scalability**: Handle high-volume, real-time data with limited resources

### When NOT to Use Streams

If your application already has all data readily available in memory, streams may introduce unnecessary overhead and complexity. Streams shine when dealing with:
- Large files (video, audio, logs)
- Network data (HTTP requests/responses)
- Real-time data feeds
- Data transformations on the fly

---

## Stream Types

Node.js provides four fundamental stream types:

### 1. Readable Streams

**Purpose**: Sources for sequential data consumption

**Common Examples**:
- `fs.createReadStream()` - reading files
- `http.IncomingMessage` - HTTP requests on server, responses on client
- `process.stdin` - standard input

**Key Events**:
- `data` - emitted when data chunk is available
- `end` - emitted when no more data to consume
- `readable` - signals data is available to read
- `error` - emitted on errors
- `close` - emitted when stream is closed

### 2. Writable Streams

**Purpose**: Destinations for sequential output

**Common Examples**:
- `fs.createWriteStream()` - writing to files
- `http.ServerResponse` - HTTP responses on server
- `process.stdout` and `process.stderr` - standard output/error

**Key Methods**:
- `.write(chunk)` - write data (returns boolean indicating backpressure)
- `.end([chunk])` - signal no more data will be written

### 3. Duplex Streams

**Purpose**: Implement both readable and writable interfaces simultaneously

**Common Examples**:
- `net.Socket` - TCP sockets
- Bidirectional communication channels

### 4. Transform Streams

**Purpose**: Duplex streams where output derives from input transformation

**Common Examples**:
- `zlib.createGzip()` - compression
- `crypto.createCipher()` - encryption
- Custom data transformations

**Key Method**: `_transform(chunk, encoding, callback)` - modify data as it passes through

---

## Working with Streams

### Method 1: `.pipe()` (Legacy)

The `.pipe()` method chains streams together:

```javascript
const fs = require('fs');

fs.createReadStream('input.txt')
  .pipe(fs.createWriteStream('output.txt'));
```

>   **PITFALL WARNING**
>
> **Manual Error Handling Required**: `.pipe()` delegates all error handling to the programmer, making it difficult to get right. Errors in any stream in the chain need separate `.on('error')` handlers on each stream, or they can crash your application.
>
> **No Cleanup**: If an error occurs mid-stream, resources may not be properly cleaned up, potentially causing memory leaks.
>
> **Recommendation**: Use `pipeline()` instead for production code.

### Method 2: `pipeline()` (Recommended)

The `pipeline()` function from `stream/promises` handles errors and cleanup automatically:

```javascript
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');
const zlib = require('node:zlib');

async function compressFile() {
  try {
    await pipeline(
      fs.createReadStream('input.txt'),
      zlib.createGzip(),
      fs.createWriteStream('input.txt.gz')
    );
    console.log('Compression complete');
  } catch (err) {
    console.error('Pipeline failed:', err);
  }
}

compressFile();
```

**Benefits**:
- Automatic error propagation
- Proper resource cleanup
- Clean async/await syntax
- Handles backpressure automatically

### Method 3: Async Iterators (Modern)

The most readable approach using `for await...of`:

```javascript
const fs = require('node:fs');

async function readFile() {
  const readable = fs.createReadStream('large-file.txt', { encoding: 'utf8' });

  try {
    for await (const chunk of readable) {
      console.log('Received chunk:', chunk.length);
      // Process chunk
    }
  } catch (err) {
    console.error('Error reading file:', err);
  }
}

readFile();
```

**Benefits**:
- Enhanced readability and cleaner code structure
- Straightforward error handling via try/catch
- Inherent backpressure management
- Works with async operations inside the loop

---

## Creating Custom Streams

### Custom Transform Stream

```javascript
const { Transform } = require('node:stream');

// Transform stream that converts text to uppercase
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    // Process the chunk
    const upperChunk = chunk.toString().toUpperCase();

    // Push transformed data downstream
    this.push(upperChunk);

    // Signal completion
    callback();
  }
});

// Usage
process.stdin
  .pipe(upperCaseTransform)
  .pipe(process.stdout);
```

### Custom Readable Stream

```javascript
const { Readable } = require('node:stream');

class CounterStream extends Readable {
  constructor(max, options) {
    super(options);
    this.max = max;
    this.current = 0;
  }

  _read(size) {
    if (this.current >= this.max) {
      this.push(null); // Signal end of stream
      return;
    }

    this.push(String(this.current++) + '\n');
  }
}

// Usage
const counter = new CounterStream(5);
counter.pipe(process.stdout);
// Outputs: 0, 1, 2, 3, 4
```

---

## Backpressure: The Critical Concept

### What is Backpressure?

Backpressure describes a buildup of data when a source produces data faster than the destination can consume it. Without proper handling, this leads to catastrophic problems.

### Why Backpressure Matters

Testing reveals dramatic impacts of ignoring backpressure:

| Metric | With Backpressure | Without Backpressure |
|--------|-------------------|----------------------|
| Memory Usage | ~88 MB | ~1.52 GB (17x more) |
| GC Frequency | 75 times/min | 36 times/min |
| GC Intensity | Low, frequent | High, aggressive |

**Real-World Consequences**:
- Memory exhaustion and OOM crashes
- System degradation affecting other processes
- Monopolized computing resources
- Degraded application performance

### How Backpressure Works

The `.write()` method returns a boolean that signals backpressure:

```javascript
const fs = require('fs');

const readable = fs.createReadStream('large-file.txt');
const writable = fs.createWriteStream('output.txt');

readable.on('data', (chunk) => {
  const canContinue = writable.write(chunk);

  if (!canContinue) {
    // Buffer is full - pause reading
    console.log('Backpressure detected, pausing...');
    readable.pause();
  }
});

writable.on('drain', () => {
  // Buffer drained - resume reading
  console.log('Buffer drained, resuming...');
  readable.resume();
});

readable.on('end', () => {
  writable.end();
});
```

**Flow**:
1. When buffer exceeds `highWaterMark` or write queue is busy, `.write()` returns `false`
2. This signals to pause the incoming Readable stream
3. Once buffer empties, `drain` event fires
4. Reading resumes, maintaining fixed memory usage

### The Golden Rules

1. **Never `.push()` unless requested** - respect the return value
2. **Never call `.write()` after it returns false** - wait for `drain` event
3. **Test implementations across Node.js versions** - behavior can vary

---

## Common Pitfalls

>   **PITFALL WARNING**
>
> ### 1. Ignoring `.write()` Return Value
>
> **Bad Practice**:
> ```javascript
> readable.on('data', data => {
>   writable.write(data); // Ignores backpressure!
> });
> ```
>
> This unconditionally pushes data regardless of destination readiness, leading to memory exhaustion.
>
> **Good Practice**:
> ```javascript
> readable.on('data', data => {
>   if (!writable.write(data)) {
>     readable.pause();
>   }
> });
>
> writable.on('drain', () => {
>   readable.resume();
> });
> ```

>   **PITFALL WARNING**
>
> ### 2. Ignoring `.push()` Return Value in Custom Readable Streams
>
> **Bad Practice**:
> ```javascript
> class MyReadable extends Readable {
>   _read(size) {
>     let chunk;
>     while (null !== (chunk = getNextChunk())) {
>       this.push(chunk); // Ignores return value!
>     }
>   }
> }
> ```
>
> Continues reading despite downstream congestion.
>
> **Good Practice**:
> ```javascript
> class MyReadable extends Readable {
>   _read(size) {
>     let chunk;
>     let canPushMore = true;
>     while (canPushMore && null !== (chunk = getNextChunk())) {
>       canPushMore = this.push(chunk);
>     }
>   }
> }
> ```

>   **PITFALL WARNING**
>
> ### 3. Multiple Callback Executions in `._write()`
>
> When implementing custom Writable streams, the callback must execute **exactly once**.
>
> **Incorrect** - multiple executions:
> ```javascript
> class MyWritable extends Writable {
>   _write(chunk, encoding, callback) {
>     if (chunk.toString().indexOf('a') >= 0) {
>       callback();
>     } else if (chunk.toString().indexOf('b') >= 0) {
>       callback();
>     }
>     callback(); // Always executes - ERROR!
>   }
> }
> ```
>
> **Correct** - single execution path:
> ```javascript
> class MyWritable extends Writable {
>   _write(chunk, encoding, callback) {
>     if (chunk.toString().indexOf('a') >= 0) {
>       return callback();
>     }
>     if (chunk.toString().indexOf('b') >= 0) {
>       return callback();
>     }
>     callback();
>   }
> }
> ```

>   **PITFALL WARNING**
>
> ### 4. Misusing `.cork()` and `.uncork()`
>
> Calling `.uncork()` multiple times without corresponding `.cork()` calls breaks write optimization.
>
> **Problematic**:
> ```javascript
> ws.cork();
> ws.write('hello ');
> ws.uncork(); // First uncork
>
> ws.cork();
> ws.write('world ');
> ws.uncork(); // Second uncork - breaks batching!
> ```
>
> **Better approach** - batch with `process.nextTick()`:
> ```javascript
> ws.cork();
> ws.write('hello ');
> ws.write('world ');
> process.nextTick(() => ws.uncork());
> ```

>   **PITFALL WARNING**
>
> ### 5. Error Handling with Multiple Streams
>
> Each stream in a chain needs error handling, or one error can crash your entire application.
>
> **Bad**:
> ```javascript
> fs.createReadStream('input.txt')
>   .pipe(zlib.createGzip())
>   .pipe(fs.createWriteStream('output.gz'));
> // No error handling!
> ```
>
> **Good** (but verbose):
> ```javascript
> const readable = fs.createReadStream('input.txt');
> const gzip = zlib.createGzip();
> const writable = fs.createWriteStream('output.gz');
>
> readable.on('error', err => console.error('Read error:', err));
> gzip.on('error', err => console.error('Gzip error:', err));
> writable.on('error', err => console.error('Write error:', err));
>
> readable.pipe(gzip).pipe(writable);
> ```
>
> **Best** - use `pipeline()`:
> ```javascript
> await pipeline(
>   fs.createReadStream('input.txt'),
>   zlib.createGzip(),
>   fs.createWriteStream('output.gz')
> );
> // Automatically handles all errors!
> ```

---

## Advanced Concepts

### Object Mode

By default, streams work with strings and Buffers. Setting `objectMode: true` allows streams to work with arbitrary JavaScript values:

```javascript
const { Transform } = require('node:stream');

const objectTransform = new Transform({
  objectMode: true,
  transform(obj, encoding, callback) {
    // Transform objects
    obj.processed = true;
    obj.timestamp = Date.now();
    this.push(obj);
    callback();
  }
});
```

### Configuring `highWaterMark`

The `highWaterMark` option controls the internal buffer size (in bytes for buffer mode, or number of objects for object mode):

```javascript
const readable = fs.createReadStream('file.txt', {
  highWaterMark: 64 * 1024 // 64KB buffer instead of default 16KB
});
```

**When to adjust**:
- Lower for memory-constrained environments
- Higher for high-throughput scenarios
- Balance between memory usage and number of I/O operations

### Web Streams Interoperability

Node.js streams differ from WHATWG Web Streams. Use conversion utilities when needed:

```javascript
const { Readable } = require('node:stream');

// Convert Node.js stream to Web Stream
const nodeStream = fs.createReadStream('file.txt');
const webStream = Readable.toWeb(nodeStream);

// Convert Web Stream to Node.js stream
const nodeStreamAgain = Readable.fromWeb(webStream);
```

---

## Interview Questions

### Conceptual Understanding

**Q1: Explain the phrase "streams are arrays over time." What does this mean?**

<details>
<summary>Answer</summary>

Both arrays and streams represent collections of data, but they differ in time dimension:
- **Arrays**: All data exists in memory simultaneously as a single complete structure
- **Streams**: Data arrives as discrete chunks over time, never existing as a complete structure in memory

Just as you iterate over array elements with `for` loops, you process stream chunks as they arrive with event handlers or async iterators. The key difference is temporal - streams handle data that hasn't fully arrived yet or is too large to fit in memory at once.
</details>

**Q2: What are the four types of streams in Node.js? Give a real-world analogy for each.**

<details>
<summary>Answer</summary>

1. **Readable**: Like a water faucet - source that produces data you consume
   - Example: Reading a file, HTTP request body

2. **Writable**: Like a drain - destination that consumes data you produce
   - Example: Writing to a file, HTTP response

3. **Duplex**: Like a phone call - simultaneous two-way communication
   - Example: TCP socket, WebSocket

4. **Transform**: Like a water filter - modifies data passing through
   - Example: Gzip compression, encryption, data parsing
</details>

**Q3: Why is backpressure critical in stream processing? What happens without it?**

<details>
<summary>Answer</summary>

Backpressure prevents memory exhaustion when a producer generates data faster than a consumer can process it.

**Without backpressure handling**:
- Memory usage can grow 17x or more
- Garbage collector works harder (from 75 GCs/min to 36 but more intensive)
- System-wide performance degradation
- Eventual out-of-memory crashes
- Monopolized CPU resources

**With proper backpressure**:
- Fixed, predictable memory usage
- The `.write()` return value signals when to pause/resume
- The `drain` event indicates readiness to continue
- Maintains sustainable throughput without overwhelming the system
</details>

### Technical Implementation

**Q4: What's the difference between `.pipe()`, `pipeline()`, and async iterators? When would you use each?**

<details>
<summary>Answer</summary>

**`.pipe()`**:
- **Use case**: Legacy code, simple scenarios
- **Pros**: Concise syntax
- **Cons**: Manual error handling required, no automatic cleanup, error-prone
- **Example**: `readable.pipe(writable)`

**`pipeline()`**:
- **Use case**: Production code, chaining multiple streams
- **Pros**: Automatic error handling, resource cleanup, backpressure management
- **Cons**: Requires async/await understanding
- **Example**: `await pipeline(readable, transform, writable)`

**Async Iterators**:
- **Use case**: When you need fine-grained control or async operations per chunk
- **Pros**: Most readable, natural control flow, easy error handling, supports async processing
- **Cons**: More verbose for simple piping
- **Example**: `for await (const chunk of readable) { ... }`

**Recommendation**: Default to `pipeline()` for most cases. Use async iterators when you need to perform async operations on each chunk.
</details>

**Q5: What does it mean when `.write()` returns `false`? What should you do?**

<details>
<summary>Answer</summary>

When `.write()` returns `false`, it means:
1. The internal buffer has exceeded `highWaterMark`
2. The write queue is busy/full
3. Backpressure is occurring

**What you should do**:
1. **Stop writing** - Do not call `.write()` again
2. **Pause the source** - If reading from a stream, call `.pause()`
3. **Wait for `drain` event** - Listen for this event on the writable stream
4. **Resume** - Once `drain` fires, it's safe to continue writing

```javascript
if (!writable.write(chunk)) {
  readable.pause();
}
writable.on('drain', () => {
  readable.resume();
});
```

**Ignoring `false`** will cause unbounded memory growth and eventual crashes.
</details>

**Q6: In a custom Readable stream's `._read()` method, why must you check the return value of `.push()`?**

<details>
<summary>Answer</summary>

The return value of `.push()` signals whether the internal buffer can accept more data:
- **`true`**: Buffer has room, safe to push more data
- **`false`**: Buffer is full (backpressure), stop pushing until `._read()` is called again

**Why it matters**:
Without checking, you'll keep pushing data into an already-full buffer, defeating backpressure and causing the same memory issues you're trying to avoid.

**Correct pattern**:
```javascript
_read(size) {
  let canPushMore = true;
  while (canPushMore && this.hasMoreData()) {
    const chunk = this.getNextChunk();
    canPushMore = this.push(chunk);
  }
}
```

When `canPushMore` becomes `false`, stop immediately. Node.js will call `._read()` again when the consumer is ready for more data.
</details>

### Practical Scenarios

**Q7: You're processing a 10GB log file to extract error lines. How would you implement this efficiently with streams?**

<details>
<summary>Answer</summary>

```javascript
const { pipeline } = require('node:stream/promises');
const { createReadStream, createWriteStream } = require('node:fs');
const { Transform } = require('node:stream');
const readline = require('node:readline');

async function extractErrors() {
  const errorFilter = new Transform({
    objectMode: true,
    transform(line, encoding, callback) {
      if (line.includes('ERROR') || line.includes('FATAL')) {
        this.push(line + '\n');
      }
      callback();
    }
  });

  const rl = readline.createInterface({
    input: createReadStream('app.log', { encoding: 'utf8' }),
    crlfDelay: Infinity
  });

  try {
    await pipeline(
      rl,
      errorFilter,
      createWriteStream('errors.log')
    );
    console.log('Error extraction complete');
  } catch (err) {
    console.error('Failed:', err);
  }
}

extractErrors();
```

**Why this works**:
- Streams process line by line, never loading the full 10GB into memory
- `pipeline()` handles backpressure automatically
- Memory usage stays constant regardless of file size
- Errors are properly propagated and handled
</details>

**Q8: Your Node.js server is processing file uploads. Users report timeouts with large files. You suspect memory issues. How do you diagnose and fix this?**

<details>
<summary>Answer</summary>

**Diagnosis**:
1. Check if uploads are buffered before processing: `req.body` suggests middleware is buffering
2. Monitor memory usage during uploads: `process.memoryUsage()`
3. Look for non-streaming file handling like `fs.readFile()` or `buffer.concat()`

**The Problem** - Likely buffering entire upload:
```javascript
// BAD - buffers entire upload in memory
app.post('/upload', (req, res) => {
  const chunks = [];
  req.on('data', chunk => chunks.push(chunk));
  req.on('end', () => {
    const file = Buffer.concat(chunks); // OOM with large files!
    fs.writeFileSync('upload.bin', file);
    res.send('OK');
  });
});
```

**The Fix** - Stream directly to disk:
```javascript
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');

app.post('/upload', async (req, res) => {
  try {
    await pipeline(
      req, // req is a readable stream
      fs.createWriteStream('./uploads/' + req.headers['filename'])
    );
    res.send('Upload complete');
  } catch (err) {
    console.error('Upload failed:', err);
    res.status(500).send('Upload failed');
  }
});
```

**Why this fixes it**:
- No buffering - data streams directly to disk
- Memory usage stays constant regardless of file size
- Backpressure automatically handled by `pipeline()`
- Proper error handling prevents hanging connections
</details>

**Q9: You have a Transform stream that performs CPU-intensive cryptographic operations. Under high load, memory usage grows unbounded. What's likely wrong and how do you fix it?**

<details>
<summary>Answer</summary>

**Root Cause**: The Transform stream's `._transform()` method is likely calling `callback()` before the async crypto operation completes, causing the stream to accept more data than it can process.

**Wrong Implementation**:
```javascript
class CryptoTransform extends Transform {
  _transform(chunk, encoding, callback) {
    // Async operation
    crypto.subtle.encrypt(algorithm, key, chunk)
      .then(encrypted => this.push(encrypted));

    callback(); // Called immediately - doesn't wait!
    // Stream continues accepting data while crypto is still processing
  }
}
```

**Correct Implementation**:
```javascript
class CryptoTransform extends Transform {
  _transform(chunk, encoding, callback) {
    crypto.subtle.encrypt(algorithm, key, chunk)
      .then(encrypted => {
        this.push(encrypted);
        callback(); // Only call after operation completes
      })
      .catch(err => callback(err));
  }
}
```

**Even Better** - Use async/await if supported:
```javascript
class CryptoTransform extends Transform {
  async _transform(chunk, encoding, callback) {
    try {
      const encrypted = await crypto.subtle.encrypt(algorithm, key, chunk);
      this.push(encrypted);
      callback();
    } catch (err) {
      callback(err);
    }
  }
}
```

**Key Principle**: Never call `callback()` until the transformation is truly complete. This is how backpressure works in Transform streams - the callback signals readiness for more data.
</details>

**Q10: Compare memory usage when processing a 1GB file with `fs.readFile()` vs streams. What would you expect?**

<details>
<summary>Answer</summary>

**With `fs.readFile()`**:
```javascript
// BAD - loads entire 1GB into memory
fs.readFile('1gb-file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  // data is 1GB+ string in memory
  processData(data);
});
```
- Memory usage: **~1GB minimum** (plus processing overhead)
- Startup time: **Slow** (must read entire file first)
- Risk: **High** (OOM with concurrent operations)

**With Streams**:
```javascript
// GOOD - processes in 64KB chunks (default highWaterMark)
const readable = fs.createReadStream('1gb-file.txt', { encoding: 'utf8' });

for await (const chunk of readable) {
  await processChunk(chunk);
}
```
- Memory usage: **~64KB** (plus processing overhead for current chunk)
- Startup time: **Immediate** (processes as data arrives)
- Risk: **Low** (fixed memory footprint)

**Real-world difference**:
- `fs.readFile()`: 1GB+ memory, blocks until complete
- Streams: 64KB memory, starts immediately, handles files of any size

**Rule of thumb**: Use `fs.readFile()` only for small files (<10MB) where you need the entire content. Use streams for anything larger or when processing data incrementally.
</details>

---

## Best Practices

1. **Always use `pipeline()` or async iterators** instead of `.pipe()` for production code
2. **Respect backpressure** - check return values of `.write()` and `.push()`
3. **Handle errors explicitly** - unhandled stream errors crash your application
4. **Call callbacks exactly once** in custom stream implementations
5. **Use object mode sparingly** - it disables important optimizations
6. **Test with large datasets** - streaming bugs often only appear under load
7. **Monitor memory usage** - use `process.memoryUsage()` to verify streaming behavior
8. **Consider `highWaterMark` tuning** for high-throughput scenarios
9. **Clean up resources** - always handle `close` and `error` events
10. **Prefer built-in streams** - use `readline`, `zlib`, `crypto` streams over reinventing

---

## Additional Resources

- [Node.js Stream Documentation](https://nodejs.org/api/stream.html)
- [Stream Handbook by substack](https://github.com/substack/stream-handbook)
- [Understanding Streams in Node.js](https://nodesource.com/blog/understanding-streams-in-nodejs)
- [`readable-stream` npm package](https://www.npmjs.com/package/readable-stream) - for cross-version compatibility
