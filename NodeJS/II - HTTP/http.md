### Anatomy of an HTTP Transaction (Node.js) — Synthesis

Source: [Node.js Learn — Anatomy of an HTTP Transaction](https://nodejs.org/en/learn/http/anatomy-of-an-http-transaction)

---

### What you’ll learn

- **Create a server** with `http.createServer` and start it with `server.listen(port)`.
- **Inspect requests**: method, URL, headers, and body using the `IncomingMessage` stream.
- **Send responses**: set status, headers, and body via the `ServerResponse` writable stream.
- **Handle routing** with simple conditionals or a router/framework.
- **Work with streams**: `request` is readable; `response` is writable; you can `pipe`.

---

### 1) Create the server

```js
const http = require('node:http');

const server = http.createServer((request, response) => {
  // handle each request here
});

server.listen(8080);
```

- The request handler runs on every incoming HTTP request.
- The server is an `EventEmitter` (you can `server.on('request', ...)`).

---

### 2) Read method, URL, headers

```js
const { method, url, headers } = request; // request is an IncomingMessage
const userAgent = headers['user-agent']; // header names are always lower-case
```

Notes:

- `url` is the path + query (no scheme/host/port).
- Repeated headers may be joined/overwritten; use `rawHeaders` if you need originals.

---

### 3) Read the request body (POST/PUT, etc.)

`request` is a readable stream. Aggregate chunks and convert to a string when `'end'` fires.

```js
let body = [];
request
  .on('data', (chunk) => body.push(chunk))
  .on('end', () => {
    body = Buffer.concat(body).toString();
    // use body here
  })
  .on('error', (err) => {
    console.error(err);
  });
```

---

### 4) Send a response: status, headers, body

```js
response.statusCode = 200;
response.setHeader('Content-Type', 'application/json');
response.end(JSON.stringify({ ok: true }));
```

Or set headers + status explicitly before body:

```js
response.writeHead(200, { 'Content-Type': 'application/json' });
response.end(JSON.stringify({ ok: true }));
```

Important: Set status and headers before writing body data.

---

### 5) Simple routing and echo server

```js
const http = require('node:http');

http
  .createServer((request, response) => {
    request.on('error', (err) => {
      console.error(err);
      response.statusCode = 400;
      response.end();
      return;
    });

    if (request.method === 'POST' && request.url === '/echo') {
      // Stream the request directly back to the client
      request.pipe(response);
    } else {
      response.statusCode = 404;
      response.end();
    }
  })
  .listen(8080);
```

Why `pipe`? Because `request` is readable and `response` is writable; this is the most direct echo.

---

### 6) Error handling (both sides are streams)

- `request` (readable) and `response` (writable) can emit `'error'`.
- Always add listeners. Unhandled stream errors can crash the process.

```js
request.on('error', (err) => {
  /* log and respond with appropriate status */
});
response.on('error', (err) => {
  /* log; connection may have closed */
});
```

---

> ⚠️ **Pitfalls**
>
> - **Missing error listeners**: Unhandled `'error'` on `request`/`response` can crash your app.
> - **Writing body before headers**: Set `statusCode`/`setHeader` or `writeHead` first.
> - **Assuming header case**: Request header names are lower-case in Node; don’t expect original case.
> - **Body parsing assumptions**: Don’t assume string data; aggregate `Buffer` chunks and convert.
> - **Ignoring backpressure**: Large payloads benefit from streaming/`pipe` rather than buffering all in memory.
> - **Naive routing**: String equality on `url` ignores querystrings and decoding; consider a router or URL parsing.
> - **Blocking the event loop**: Heavy CPU work will delay I/O. Offload CPU-bound tasks if needed.

---

### Interview-ready talking points

- **Server lifecycle**: Creating with `createServer`, listening with `listen`, per-request handler.
- **Request parsing**: Accessing method, URL, headers; reading the body via stream events.
- **Response composition**: Status, headers, body; difference between implicit headers and `writeHead`.
- **Routing approaches**: Conditionals vs. routers vs. frameworks; why and when to choose each.
- **Streams**: Using `pipe`, handling backpressure, avoiding full-buffer reads for large payloads.
- **Error handling**: Where errors can occur, consequences of missing listeners, graceful responses.

---

### Quick checklist

- [ ] Did you add `'error'` handlers for `request` and `response`?
- [ ] Did you set `statusCode` and headers before writing the body?
- [ ] Are you safely reading the body as `Buffer` chunks?
- [ ] Is routing correct for method + path (and queries, if relevant)?
- [ ] Could you stream or `pipe` instead of buffering large bodies?

---

### Reference

- Node.js Learn — Anatomy of an HTTP Transaction: [link](https://nodejs.org/en/learn/http/anatomy-of-an-http-transaction)
