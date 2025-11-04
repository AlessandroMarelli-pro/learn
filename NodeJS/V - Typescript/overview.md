### TypeScript in Node.js — Overview

Sources: [Introduction](https://nodejs.org/en/learn/typescript/introduction) · [Run Natively](https://nodejs.org/en/learn/typescript/run-natively) · [Transpile](https://nodejs.org/en/learn/typescript/transpile) · [Run with a runner](https://nodejs.org/en/learn/typescript/run) · [Publish a TS package](https://nodejs.org/en/learn/typescript/publishing-a-ts-package)

---

### What and why
- **TypeScript** adds static types on top of JavaScript for better tooling, early error detection, and maintainability.
- TS consists of: (1) JS + type syntax, (2) type definitions (`.d.ts`), often from `@types/*` (e.g., `@types/node`).

```ts
type User = { name: string; age: number };
function isAdult(user: User): boolean { return user.age >= 18; }
```

---

### Running TS in Node.js

1) Natively (Node v22+ experimental → type stripping; v22.18+ on by default)

```bash
# v22.6+: strip types
node --experimental-strip-types app.ts

# v22.7+: transform TS-only syntax (also implies strip-types)
node --experimental-transform-types app-with-enum.ts

# v22.18+: type stripping enabled by default
node app.ts
```

- Transform-only syntax (e.g., `enum`, `namespace`) still requires `--experimental-transform-types`.
- Amaro loader doesn’t use `tsconfig.json` to run; configure your editor/`tsc` separately.

2) Transpile first (stable, portable)

```bash
npm i -D typescript
npx tsc app.ts
node app.js
```

3) Use a runner (dev convenience)

```bash
# ts-node
npm i -D ts-node typescript
npx ts-node app.ts

# tsx
npm i -D tsx typescript
npx tsx app.ts
# or register with node
node --import=tsx app.ts
```

---

### Types and definitions
- Install Node types for API intellisense and checks:

```bash
npm i -D @types/node
```

```ts
import { resolve } from 'node:path';
// resolve(123) // TS error: number not assignable to string
```

---

### Publishing a TypeScript package (high-level)
- Emit JavaScript plus types for consumers:
  - Build JS output (e.g., `dist/`) and ship `.d.ts` (generated or authored).
  - Set `package.json`: `types` → path to bundled `.d.ts`; `exports` map JS files.
  - Ensure `tsconfig.json` emits declaration files (`declaration: true`) and compatible module targets.

---

> ⚠️ **Pitfalls**
>
> - **Assuming native TS runs everything**: Transform-only syntax still needs `--experimental-transform-types` (until stabilized).
> - **Missing type defs**: Forgetting `@types/node` (and other `@types/*`) leads to poor DX and loose checks.
> - **Mismatched module settings**: `module`, `moduleResolution`, and `type` (module type) must align with Node’s ESM/CJS behavior.
> - **Publishing without declarations**: Consumers lose types; configure `declaration: true` and include `.d.ts` in publish.
> - **Relying on `tsconfig` for native run**: Node’s loader ignores it; configure tools separately.

---

### Interview-ready talking points
- Compare native TS (type stripping) vs transpilation vs runners; trade-offs and when to use each.
- How `@types/*` definitions integrate with JS libraries and Node core APIs.
- ESM/CJS considerations: `type: module`, `exports`, and TS `module` settings.
- Publishing TS: emitting declarations, export maps, and compatibility for consumers.

---

### Quick checklist
- [ ] Do I need native run, a runner, or a build step (prod)?
- [ ] Are Node and library types installed (`@types/node`, etc.)?
- [ ] Is my module format (ESM/CJS) consistent across Node, TS, and `package.json`?
- [ ] For packages, are `.d.ts` files generated and referenced via `types`?


