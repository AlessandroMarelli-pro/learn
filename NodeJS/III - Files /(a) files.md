### Working with Files in Node.js — Synthesis

Sources: [File stats](https://nodejs.org/en/learn/manipulating-files/nodejs-file-stats) · [File paths](https://nodejs.org/en/learn/manipulating-files/nodejs-file-paths) · [Reading files](https://nodejs.org/en/learn/manipulating-files/reading-files-with-nodejs) · [Writing files](https://nodejs.org/en/learn/manipulating-files/writing-files-with-nodejs) · [File descriptors](https://nodejs.org/en/learn/manipulating-files/working-with-file-descriptors-in-nodejs) · [Working with folders](https://nodejs.org/en/learn/manipulating-files/working-with-folders-in-nodejs)

---

### Overview

- **fs**: core module for file I/O (callbacks, sync, and promises APIs).
- **path**: utilities for cross-platform-safe path manipulation.
- **Streams**: read/write large files efficiently without loading into memory.
- **Descriptors**: low-level handles to open files for advanced control.
- **Folders**: create, read, and delete directories recursively and safely.

---

### File stats (metadata)

- Use `fs.stat(path, cb)` / `fs.statSync(path)` / `fs.promises.stat(path)`.
- Common checks: `stats.isFile()`, `stats.isDirectory()`, `stats.isSymbolicLink()`, `stats.size`.

```js
const fs = require('node:fs/promises');

const stats = await fs.stat('/path/to/file');
if (stats.isFile()) {
  console.log('bytes:', stats.size);
}
```

Reference: [File stats](https://nodejs.org/en/learn/manipulating-files/nodejs-file-stats)

---

### File paths (portable path handling)

- Import: `const path = require('node:path');`
- Extract parts: `path.dirname(p)`, `path.basename(p)`, `path.extname(p)`.
- Build paths: `path.join(...)` (normalizes separators), `path.resolve(...)` (absolute path).

```js
const path = require('node:path');

const file = path.join(process.cwd(), 'logs', 'app.log');
const nameWithoutExt = path.basename(file, path.extname(file));
```

Reference: [File paths](https://nodejs.org/en/learn/manipulating-files/nodejs-file-paths)

---

### Reading files

- Entire file (simple, memory-heavy): `fs.readFile`, `fs.readFileSync`, `fs.promises.readFile`.
- Large files: prefer streams (`fs.createReadStream`) to reduce memory pressure.

```js
const fs = require('node:fs');

const stream = fs.createReadStream('big.txt', { encoding: 'utf8' });
for await (const chunk of stream) {
  // process chunk
}
```

Reference: [Reading files](https://nodejs.org/en/learn/manipulating-files/reading-files-with-nodejs)

---

### Writing files

- Entire content: `fs.writeFile`, `fs.writeFileSync`, `fs.promises.writeFile`.
- Append: `fs.appendFile` / `fs.promises.appendFile`.
- Streams for large writes: `fs.createWriteStream`.

```js
const fs = require('node:fs/promises');

await fs.writeFile('report.json', JSON.stringify({ ok: true }), 'utf8');
```

Reference: [Writing files](https://nodejs.org/en/learn/manipulating-files/writing-files-with-nodejs)

---

### Working with file descriptors (low-level)

- Open/close: `fs.open(path, flags[, mode])` → `FileHandle` (promises) or fd (callbacks).
- Read/write by position, control flags, fine-grained resource management.
- Always close descriptors to avoid resource leaks.

```js
const fs = require('node:fs/promises');

const fh = await fs.open('data.bin', 'r');
try {
  const { bytesRead, buffer } = await fh.read({
    buffer: Buffer.alloc(1024),
    position: 0,
  });
  // use buffer.slice(0, bytesRead)
} finally {
  await fh.close();
}
```

Reference: [File descriptors](https://nodejs.org/en/learn/manipulating-files/working-with-file-descriptors-in-nodejs)

---

### Working with folders (directories)

- Create: `fs.mkdir(path, { recursive: true })`.
- List: `fs.readdir(path, { withFileTypes: true })` to get types via `Dirent`.
- Remove: `fs.rm(path, { recursive: true, force: true })` or `fs.rmdir` (non-recursive).

```js
const fs = require('node:fs/promises');

await fs.mkdir('output/logs', { recursive: true });
const entries = await fs.readdir('output', { withFileTypes: true });
const files = entries.filter((e) => e.isFile()).map((e) => e.name);
```

Reference: [Working with folders](https://nodejs.org/en/learn/manipulating-files/working-with-folders-in-nodejs)

---

> ⚠️ **Pitfalls**
>
> - **Blocking sync APIs**: `fs.*Sync` blocks the event loop; avoid on servers’ hot paths.
> - **Reading huge files into memory**: Prefer `createReadStream`/`pipeline` to stream data.
> - **Path portability**: Don’t concatenate with `'/'`; use `path.join` for cross-platform safety.
> - **Unchecked paths**: `path.resolve`/`normalize` don’t verify existence; still validate and handle errors.
> - **Descriptor leaks**: Always `close()` file descriptors/handles in `finally` blocks.
> - **Race conditions**: Existence checks (`stat`) followed by operations can race; handle `ENOENT`/`EEXIST` errors.
> - **Encoding mistakes**: Specify encoding (`'utf8'`) for text; omit for binary to get `Buffer`s.
> - **Permissions/flags**: Choose correct flags (e.g., `'wx'` to avoid overwriting existing files).
> - **Recursive deletes**: `rm({ recursive: true })` is powerful—double-check the target path.

---

### Interview-ready talking points

- **fs APIs**: Differences between callbacks, sync, and promises; when to use each.
- **Streams vs readFile**: Memory/performance trade-offs for large files.
- **Path handling**: `join`, `resolve`, `dirname`/`basename`/`extname`, platform separators.
- **File descriptors**: Why/when to use low-level operations; resource lifecycle (open → use → close).
- **Dir ops**: Recursive creation, listing with `Dirent`, safe deletion patterns.
- **Error handling**: Common `errno` codes (`ENOENT`, `EEXIST`, `EPERM`), retry/backoff patterns.

---

### Quick checklist

- [ ] Is this path portable (built with `path.join`/`resolve`)?
- [ ] For large files, am I using streams instead of buffering everything?
- [ ] Are encodings correct for text/binary?
- [ ] Do I close descriptors/handles reliably in `finally`?
- [ ] Do I handle common `fs` errors (existence, permissions, races)?
