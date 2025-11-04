### Memory Diagnostics in Node.js — Synthesis

Source: [Memory](https://nodejs.org/en/learn/diagnostics/memory)

---

### What to recognize

- **Runaway memory (leak)**: RSS/heap grows over time (minutes → weeks), GC gets busier, process slows, eventually OOM/restarts.
- **Inefficient usage**: Unexpectedly high memory or frequent GC even without growth.

Typical side effects:

- Restarts drop requests; increased GC CPU; potential swapping; snapshots may fail when memory is critically low.

---

### Core goal

Find: (1) which objects dominate memory, (2) who retains them (retainer paths), (3) allocation patterns over time.

---

### Toolbox (high level)

- **Heap Profiler**: record allocations over time; identify hot allocation sites.
- **Heap Snapshots**: capture object graph; inspect dominators/retainers and sizes.
- **GC Traces**: observe GC frequency/duration to infer pressure.

---

### Practical workflow (minimal commands)

1. Start with the Inspector (local or remote)

```bash
node --inspect index.js
# or, bind to a host/port explicitly
node --inspect=0.0.0.0:9229 index.js
```

Connect Chrome DevTools → Memory tab:

- Take a **Heap Snapshot** at baseline, then after traffic → compare.
- Use **Allocation sampling** to find hot allocation stacks.

2. Force and observe GC (diagnostics only)

```bash
node --expose-gc index.js
# inside code or REPL
global.gc();
```

3. Enable GC trace logs when needed

```bash
node --trace-gc --trace-gc-verbose index.js
```

Interpretation hints:

- Rising retained size of a class/array/map across snapshots → leak suspect.
- Retainer path points to the object keeping it alive (e.g., global cache, LRU bug, event listener).

---

### Common leak sources to check

- Caches/Maps/Sets without eviction or with unbounded keys.
- Never-removed event listeners/Intervals/Timeouts.
- Request-scoped data stored in process-wide structures.
- Large buffers/strings accumulated by batching instead of streaming.

---

> ⚠️ **Pitfalls**
>
> - **Diagnosing at OOM**: Snapshots may fail when memory is exhausted; capture earlier under load.
> - **Confusing RSS with heap**: Native add-ons, buffers, and mmap can inflate RSS; inspect V8 heap directly.
> - **Relying on GC to fix logic bugs**: GC won’t free objects still referenced by retainers.
> - **No load parity**: Reproducing leaks without similar traffic patterns can hide issues.
> - **Single snapshot conclusions**: Always compare at least two snapshots to see growth and suspects.

---

### Interview-ready talking points

- Difference between a leak vs high churn: growth in retained size vs frequent but reclaimed allocations.
- How to use heap snapshots to find dominators and retainer paths.
- Interpreting GC traces (frequency/duration) and their impact on latency.
- Strategies: streaming over buffering, bounded caches, cleaning listeners/intervals, pooling.

---

### Quick checklist

- [ ] Captured baseline and post-load heap snapshots and compared dominators.
- [ ] Reviewed allocation sampling to locate hot sites.
- [ ] Validated GC behavior with `--trace-gc`.
- [ ] Audited caches/listeners/timers and large Buffer usage.
- [ ] Re-tested under realistic load to confirm fix.
