# Performance Diagnostics with Linux Perf and Flame Graphs

## Overview

Performance profiling in Node.js requires understanding CPU usage across JavaScript, native, and OS-level frames. Two powerful tools help diagnose performance bottlenecks:

- **Linux Perf**: Low-level CPU profiling tool that captures system-wide performance data
- **Flame Graphs**: Visual representations of CPU time consumption by function calls

Together, these tools enable developers to identify and resolve synchronous performance issues in Node.js applications.

---

## Linux Perf

### What is Linux Perf?

Linux Perf is a performance analysis tool built into the Linux kernel that provides detailed CPU profiling across multiple layers of the application stack. It captures data from:
- JavaScript execution
- Native C++ bindings
- OS-level system calls

### Installation

```bash
# On Ubuntu/Debian
sudo apt-get install linux-tools-common linux-tools-generic

# Verify installation
perf --version
```

### Using Perf with Node.js

Node.js must be started with specific flags to enable perf profiling:

**Production-safe flag (recommended):**
```bash
node --perf-basic-prof-only-functions app.js
```

**Full profiling flag:**
```bash
node --perf-basic-prof app.js
```

### Recording Performance Data

Once your Node.js application is running, capture profiling data:

```bash
# Find the process ID
ps aux | grep node

# Record at 99 Hz sampling rate
sudo perf record -F 99 -p [PID] -g

# For a time-limited capture (e.g., 30 seconds)
sudo perf record -F 99 -p [PID] -g -- sleep 30
```

**Key parameters:**
- `-F 99`: Sample at 99 Hz (99 times per second)
- `-p [PID]`: Target specific process
- `-g`: Enable call-graph recording

### Analyzing Results

Extract the recorded data:

```bash
# Generate human-readable output
sudo perf script > perfs.out

# View interactive report
sudo perf report
```

>   **PITFALL WARNING**
>
> **Infinite Disk Growth**: The `--perf-basic-prof` flag continuously writes to `/tmp/perf-PID.map` without cleanup, potentially filling your disk. Always use `--perf-basic-prof-only-functions` in production, or use the `linux-perf` npm module to manage file lifecycle.
>
> **Optimization Artifacts**: V8's JIT compiler optimizations can obscure function names in perf output. Node.js 10+ with `--interpreted-frames-native-stack` helps mitigate this issue.

---

## Flame Graphs

### What are Flame Graphs?

Flame graphs are interactive SVG visualizations that display:
- **Width**: Time spent in each function (wider = more CPU time)
- **Height**: Call stack depth (top functions called by bottom functions)
- **Color**: Typically orange/red for CPU-intensive operations

They excel at identifying "hot paths" - the code consuming the most CPU time.

### Generation Methods

#### Method 1: Using 0x (Simplest)

The `0x` package provides the easiest flame graph generation:

```bash
npm install -g 0x
0x app.js

# Or profile an existing process
0x -p [PID]
```

This automatically generates an interactive HTML flame graph.

#### Method 2: Manual Generation with Perf

For more control or when `0x` isn't available:

```bash
# 1. Run Node.js with perf profiling
node --perf-basic-prof --interpreted-frames-native-stack app.js &
PID=$!

# 2. Record perf data
sudo perf record -F 99 -p $PID -g -- sleep 30

# 3. Extract to text format
sudo perf script > perfs.out

# 4. Clone Brendan Gregg's FlameGraph tools
git clone https://github.com/brendangregg/FlameGraph.git

# 5. Generate the flame graph
cat perfs.out | ./FlameGraph/stackcollapse-perf.pl | \
  ./FlameGraph/flamegraph.pl --colors=js > flamegraph.svg
```

#### Method 3: Profiling Running Production Processes

Capture a snapshot without restarting:

```bash
# Profile the most recent node process for 3 seconds
sudo perf record -F 99 -p `pgrep -n node` -g -- sleep 3

# Then generate flame graph as above
sudo perf script > perfs.out
# ... continue with stackcollapse and flamegraph.pl
```

### Node.js Flags for Accurate Profiling

| Flag | Purpose | When to Use |
|------|---------|-------------|
| `--perf-basic-prof` | Full profiling with all symbols | Development/debugging |
| `--perf-basic-prof-only-functions` | Lighter output, function-level only | Production environments |
| `--interpreted-frames-native-stack` | Fixes Turbofan symbol issues | Node.js 10+ (essential) |

### Interpreting Flame Graphs

1. **Width Analysis**: Wide bars indicate functions consuming significant CPU time
2. **Color Coding**: Orange/red typically highlights CPU-intensive operations
3. **Interactive Navigation**: Click bars to zoom into specific call stacks
4. **Top-Down Reading**: Bottom functions call those above them
5. **Plateaus**: Flat tops indicate leaf functions (no further calls)

### Filtering Noise

Remove Node.js internal functions for clearer analysis:

```bash
sed -i -r \
  -e "/( __libc_start| LazyCompile | v8::internal::| Builtin:| Stub:)/d" \
  perfs.out
```

This strips out:
- C library startup code
- V8 lazy compilation
- V8 internal functions
- Built-in and stub functions

>   **PITFALL WARNING**
>
> **Node.js 8.x Turbofan Issue**: V8's Turbofan optimizer in Node.js 8 replaces optimized function names with generic `BytecodeHandler:` labels, making flame graphs nearly useless. **Solution**: Upgrade to Node.js 10+ and use `--interpreted-frames-native-stack`.
>
> **Mangled Symbols**: Some Linux distributions ship perf without proper C++ symbol demangling. Install `binutils` or update perf if you see garbled function names like `_ZN4v8...`.
>
> **Sampling Limitations**: Perf uses sampling, not tracing. Very fast functions called infrequently may not appear in results. Increase `-F` value (e.g., `-F 999`) for finer granularity, at the cost of higher overhead.
>
> **Async Operations**: Flame graphs only show synchronous CPU usage. They won't reveal I/O waits, promise chains, or event loop delays. Use async profiling tools (like `clinic.js bubbleprof`) for async bottlenecks.

---

## Workflow: End-to-End Performance Analysis

1. **Identify the problem**: Notice slow response times or high CPU usage
2. **Start profiling**: Launch Node.js with `--perf-basic-prof-only-functions`
3. **Capture data**: Use `perf record` during the problematic workload
4. **Generate flame graph**: Convert perf output to visual format
5. **Analyze**: Identify the widest bars (most CPU time)
6. **Optimize**: Refactor or optimize the hot functions
7. **Verify**: Re-profile to confirm improvements

---

## Interview Questions

### Conceptual Understanding

**Q1: What is the primary difference between Linux Perf and application-level profilers like Node.js's built-in `--prof` flag?**

<details>
<summary>Answer</summary>

Linux Perf operates at the OS/kernel level and captures data across the entire stack (JavaScript, native C++ modules, system calls), while `--prof` focuses only on V8 JavaScript execution. Perf provides a more complete picture but requires root privileges and Linux-specific tooling.
</details>

**Q2: Why do flame graphs show width rather than just call frequency?**

<details>
<summary>Answer</summary>

Width represents cumulative time (including child function calls), which is more meaningful for performance optimization. A function called once that runs for 10 seconds is more important to optimize than a function called 1000 times that runs for 1ms total. Width = impact on overall performance.
</details>

**Q3: What type of performance issues can flame graphs NOT help you diagnose?**

<details>
<summary>Answer</summary>

Flame graphs only show synchronous CPU usage. They cannot reveal:
- Asynchronous I/O bottlenecks
- Network latency issues
- Database query delays
- Event loop blocking (they show what's blocking, but not the waiting)
- Memory leaks or garbage collection pauses

For these issues, use tools like `clinic.js`, async profilers, or memory profilers.
</details>

### Technical Implementation

**Q4: You run `node --perf-basic-prof app.js` on a production server. After a week, the server runs out of disk space. What went wrong and how do you fix it?**

<details>
<summary>Answer</summary>

`--perf-basic-prof` continuously writes symbol maps to `/tmp/perf-PID.map` without cleanup, causing infinite disk growth.

**Fix**: Use `--perf-basic-prof-only-functions` instead, which produces much less output, or use the `linux-perf` npm module to manage file lifecycle automatically. Always monitor `/tmp` disk usage when profiling in production.
</details>

**Q5: Your flame graph shows mostly `BytecodeHandler:` labels instead of function names. What's the cause and solution?**

<details>
<summary>Answer</summary>

**Cause**: Node.js 8.x with V8's Turbofan optimizer replaces optimized function names with generic bytecode handler labels.

**Solution**:
1. Upgrade to Node.js 10+
2. Use the `--interpreted-frames-native-stack` flag when starting Node.js
3. This flag preserves function name information even after optimization
</details>

**Q6: Explain the `-F 99` flag in `perf record -F 99`. Why 99 instead of 100?**

<details>
<summary>Answer</summary>

`-F 99` sets the sampling frequency to 99 Hz (99 samples per second).

**Why 99 instead of 100?** Using a prime or odd number helps avoid accidental synchronization with periodic system tasks or application loops that might run at even intervals (e.g., 10ms = 100 Hz, 1ms = 1000 Hz). This prevents aliasing and ensures more representative sampling across different execution patterns.

Higher values (e.g., `-F 999`) provide finer granularity but increase profiling overhead.
</details>

### Practical Scenarios

**Q7: A developer runs perf profiling and sees their custom function `processData()` consuming 60% of CPU time. They optimize it heavily, reducing its execution time by 80%. After deploying, overall application performance only improves by 5%. What happened?**

<details>
<summary>Answer</summary>

The flame graph showed CPU time, not wall-clock time. The application likely spends most of its time waiting for I/O (database, network, file system), not in CPU-intensive operations. The 60% CPU time was only 60% of the ~10% of time spent actively using CPU.

**Lesson**: Always measure end-to-end latency alongside CPU profiling. Use async profilers to understand I/O wait times.
</details>

**Q8: You need to profile a Node.js microservice in production without restarting it. What's the safest approach?**

<details>
<summary>Answer</summary>

```bash
# 1. Find the process ID
PID=$(pgrep -n node)

# 2. Attach perf for a limited time window (e.g., 30 seconds)
sudo perf record -F 99 -p $PID -g -- sleep 30

# 3. Generate flame graph
sudo perf script > perfs.out
cat perfs.out | ./stackcollapse-perf.pl | ./flamegraph.pl > profile.svg
```

This attaches to the running process without restart, captures data for only 30 seconds (limiting overhead), and detaches automatically. Always test profiling overhead in staging first, as `-F 99` typically adds 1-3% CPU overhead.
</details>

**Q9: How would you diagnose a situation where your Node.js application shows low CPU usage (~10%) but is responding slowly to requests?**

<details>
<summary>Answer</summary>

Low CPU + slow response indicates I/O or event loop blocking issues, not CPU bottlenecks. Flame graphs won't help here.

**Better tools**:
- `clinic.js bubbleprof`: Visualizes async operations and event loop delays
- `--trace-warnings`: Detects promise rejection issues
- `async_hooks`: Custom async operation tracking
- APM tools (New Relic, DataDog): Track external service latency
- Check database query times and connection pool exhaustion

**Quick check**: Run `node --trace-warnings app.js` and monitor for unhandled rejections or long-running promises.
</details>

---

## Best Practices

1. **Always use time-limited captures** in production (`-- sleep N`)
2. **Start with low sampling rates** (`-F 99`) and increase only if needed
3. **Combine multiple diagnostic tools**: perf for CPU, clinic.js for async, heapdump for memory
4. **Profile representative workloads**: Capture during peak traffic or specific problem scenarios
5. **Filter noise aggressively**: Remove V8/libc internals unless debugging Node.js itself
6. **Annotate flame graphs**: Document what workload was running during capture
7. **Version control your profiling scripts**: Maintain consistent methodology across teams

---

## Additional Resources

- [Brendan Gregg's Flame Graph Repository](https://github.com/brendangregg/FlameGraph)
- [0x Flame Graph Tool](https://github.com/davidmarkclements/0x)
- [Node.js Official Diagnostics Guide](https://nodejs.org/en/learn/diagnostics)
- [Linux Perf Examples](http://www.brendangregg.com/perf.html)
