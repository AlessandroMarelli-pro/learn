### Resource Management & Internationalization — Teacher’s Synthesis

Sources: [Resource management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Resource_management), [Internationalization](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Internationalization)

### 1) Resource management — deterministic cleanup for non‑memory resources

- **What**: Managing external resources that JS GC does not automatically clean up (file handles, sockets, stream readers, locks, DB connections). See MDN: Resource management.
- **Goals**: Ensure resources are released on success, failure, and early returns; reduce boilerplate; compose multiple disposables safely.

#### using / await using (disposables)

- `using` binds a resource that has a synchronous `Symbol.dispose` method; it is disposed automatically at the end of scope.
- `await using` binds a resource with `Symbol.asyncDispose` for async cleanup.

```js
class Lock {
  constructor() { this.released = false; }
  [Symbol.dispose]() { this.released = true; }
}

function doWork() {
  using lock = new Lock();
  // use lock
} // lock disposed here, even on throw
```

#### DisposableStack / AsyncDisposableStack

- Stack‑like aggregators to manage multiple resources, ensuring all are disposed (LIFO), even if acquisition fails mid‑way.

```js
function processTwo(stream1, stream2) {
  const stack = new DisposableStack();
  const r1 = stack.use(stream1.getReader());
  const r2 = stack.use(stream2.getReader());
  // work with r1, r2
  stack.dispose();
}
```

#### Error handling

- Prefer `using`/stacks over manual nested `try/finally`. If manual, ensure each acquire has its own `try/finally` and handle errors in disposal without masking the original error.
- `FinalizationRegistry` is a last resort; finalizers are not guaranteed to run promptly or at all.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Relying on GC/finalizers for critical resources is unsafe; always dispose deterministically.
> - Manual multi‑resource `try/finally` can leak if acquisition or disposal throws; prefer `DisposableStack`/`AsyncDisposableStack`.
> - Inconsistent release APIs (`close`, `releaseLock`, `disconnect`) — standardize via wrappers that implement `Symbol.dispose`/`Symbol.asyncDispose`.
> - Async cleanup forgotten: use `await using` for resources needing async teardown (e.g., network/streams).
> - Hidden resource reuse after dispose: keep disposed handles out of scope or set to `null` after release.

### 2) Internationalization (i18n) — locale‑aware formatting and parsing

- **Intl core**: Locale negotiation and formatting via `Intl.*` APIs. Provide user or app locales, not the host default, to avoid surprises. See MDN: Internationalization.
- **Common formatters**:
  - `Intl.NumberFormat` (numbers, percents, currency)
  - `Intl.DateTimeFormat` (dates/times, time zones, calendars)
  - `Intl.Collator` (locale sorting/compare)
  - `Intl.RelativeTimeFormat` (e.g., “in 3 days”)
  - `Intl.ListFormat` (lists like “A, B, and C”)
  - `Intl.PluralRules` (plural category selection)
  - `Intl.DisplayNames`, `Intl.Locale`

```js
const nf = new Intl.NumberFormat('fr-FR', {
  style: 'currency',
  currency: 'EUR',
});
nf.format(1234.56); // "1 234,56 €"

const df = new Intl.DateTimeFormat('en-GB', {
  dateStyle: 'long',
  timeZone: 'Europe/London',
});
df.format(new Date());

const coll = new Intl.Collator('sv-SE');
['ö', 'z', 'ä'].sort(coll.compare); // locale‑aware order
```

- **Performance**: Reuse formatter instances; constructing per call is costly.
- **BCP 47 locales**: Tags like `en-US`, extensions like `nu-latn`, `ca-gregory`; create with `new Intl.Locale('en-US-u-nu-latn')`.
- **Time zones**: Always set `timeZone` to avoid host defaults; beware DST transitions.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Calling `toLocaleString()` without explicit locales/options yields host‑dependent results; pass locales and options.
> - Not setting `timeZone` causes inconsistent times across users/servers; DST can shift wall‑clock hours.
> - Creating `Intl.*` instances in hot paths is slow; cache and reuse.
> - String comparison without `Intl.Collator` breaks for many languages (e.g., Swedish Å/Ä/Ö).
> - Hard‑coding currency symbols or decimal/grouping separators is brittle; use `Intl.NumberFormat`.
> - RTL scripts and directionality: ensure UI mirroring and use Unicode bidi controls when concatenating user text.

### Interview‑Style Questions (Practice)

- Resource management: Compare `using`/`await using` with manual `try/finally`. When would you choose `DisposableStack`?
- How would you wrap a non‑disposable API (e.g., `reader.releaseLock`) to work with `using`? Show a tiny adapter.
- Why is `FinalizationRegistry` discouraged for critical cleanup? What guarantees are missing?
- Intl: Why prefer `Intl.Collator` over `localeCompare` or plain string `<`?
- How do you ensure stable numeric/date formatting across environments? Show options you’d set.
- What’s the impact of time zones and DST on `DateTimeFormat` and scheduling?
- Performance: How would you cache `Intl.*` instances in a formatting utility?
- Given `en-US` vs `fr-FR`, show currency and list formatting differences and how you’d test them.

### References

- MDN — Resource management: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Resource_management
- MDN — Internationalization: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Internationalization
