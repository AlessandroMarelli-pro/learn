### Node.js Streams — Synthesis

Sources: [How to use streams](https://nodejs.org/en/learn/modules/how-to-use-streams) · [Backpressuring in Streams](https://nodejs.org/en/learn/modules/backpressuring-in-streams)

---

### Why streams

- Process large data incrementally (chunks), reducing memory and improving responsiveness.
- Fit Node’s event-driven model; streams are `EventEmitter`s.
- Types: **Readable**, **Writable**, **Duplex**, **Transform**.

---

### Core patterns

- Events on readable: `data`, `readable`, `end`, `error`, `close`.
- Events on writable: `drain`, `finish`, `error`, `close`.

```js
const fs = require('node:fs');

const read = fs.createReadStream('input.txt');
read.on('data', (chunk) => {
  /* handle chunk */
});
read.on('end', () => {
  /* done */
});
read.on('error', console.error);
```

---

### Pipe vs pipeline

- `.pipe()` connects streams and handles backpressure automatically, but manual error handling is required for each stream.
- `stream.pipeline(...streams, cb)` (or promises version) wires streams with safe error/cleanup handling.

```js
const fs = require('node:fs');
const { Transform, pipeline } = require('node:stream');

const upper = new Transform({
  transform(chunk, enc, cb) {
    this.push(chunk.toString().toUpperCase());
    cb();
  },
});

pipeline(
  fs.createReadStream('input.txt'),
  upper,
  fs.createWriteStream('out.txt'),
  (err) => {
    if (err) console.error('Pipeline error:', err);
  },
);
```

---

### Async iterators (preferred consumption)

- Readables are async-iterable; clear flow, natural error handling.

```js
const fs = require('node:fs');
const { pipeline } = require('node:stream/promises');

await pipeline(
  fs.createReadStream('input.txt'),
  async function* (source) {
    for await (const chunk of source) {
      yield chunk.toString().toUpperCase();
    }
  },
  process.stdout,
);
```

---

### Object mode

- Enable `{ objectMode: true }` to push arbitrary JS values (not bytes). `highWaterMark` counts objects, not bytes.

---

### Backpressure 101

- When a writable can’t keep up, `.write()` returns `false`. Producers must pause until `'drain'` fires.
- Readables pause/resume automatically with `.pipe()`/`pipeline()`. Manual wiring requires checking return values.

```js
function writeMany(writable, data) {
  for (const chunk of data) {
    if (!writable.write(chunk)) {
      writable.once('drain', () => writeMany(writable, []));
      return; // wait for drain, then resume (example keeps it simple)
    }
  }
  writable.end();
}
```

Key idea: slow consumers signal producers to slow down; honoring this prevents memory bloat.

---

> ⚠️ **Pitfalls**
>
> - **Ignoring errors**: With `.pipe()`, attach `error` handlers or prefer `pipeline()` which handles teardown.
> - **Overlooking backpressure**: Writing blindly without checking `.write()` return value can blow up memory.
> - **Buffering entire files**: Use streams for large payloads; avoid `readFile`/`writeFile` for huge data.
> - **Forgetting `end()`**: Writable streams need `end()` when done; otherwise, processes may hang.
> - **Wrong `highWaterMark` assumptions**: It’s bytes by default; in object mode it’s item count.
> - **Mixing Web Streams and Node streams**: Different APIs—convert with `toWeb`/`fromWeb` when needed.

---

### Interview-ready talking points

- Explain stream types and when to use Transform vs Duplex.
- Compare `.pipe()` vs `pipeline()`; why `pipeline()` is safer.
- Describe backpressure: how `.write()`/`drain` coordinate producer/consumer speed.
- Show async iterator consumption and why it’s recommended.
- Discuss object mode and `highWaterMark` semantics.

---

### Quick checklist

- [ ] Use `pipeline()` for robust piping and error cleanup.
- [ ] Honor backpressure (`.write()`/`drain`) or rely on `.pipe()` semantics.
- [ ] Prefer async iterators for readability and control.
- [ ] Pick appropriate `highWaterMark` (bytes vs objects).
- [ ] Convert between Node and Web Streams explicitly when needed.
