### Working with Different Filesystems — Synthesis

Source: [How to work with Different Filesystems](https://nodejs.org/en/learn/manipulating-files/working-with-different-filesystems)

---

### Big idea

Filesystems vary (case sensitivity/preservation, Unicode normalization, timestamps, permissions, extended attributes). Don’t infer behavior from `process.platform`; probe behavior and preserve user data exactly as the filesystem stores it.

---

### Principles

- **Probe, don’t assume**: Different mounts on the same OS can behave differently.
- **Superset, not lowest common denominator**: Preserve features (case, Unicode form, timestamp resolution, permissions) and downsample only when interacting with a lesser FS.
- **Preserve on read/write**: Store and transmit filenames/timestamps unchanged; use normalization only for comparisons.
- **Match comparisons to FS behavior**: Use appropriate case/Unicode/timestamp comparison strategies for the active FS.

---

### Case preservation vs sensitivity

- Preservation: FS stores `AbC` and returns `AbC`.
- Sensitivity: FS distinguishes `abc` from `ABC`. Some FSes are insensitive (treat as equal) yet may preserve original case.

---

### Unicode form preservation vs insensitivity

- Preservation: Keep the exact byte sequence (e.g., NFC vs NFD) returned by the FS.
- Insensitivity: Compare using normalization, but do not rewrite stored data.

```js
// Comparison only — never store the normalized value
function unicodeEqual(a, b) {
  return a.normalize('NFC') === b.normalize('NFC');
}
```

Note: `String.prototype.normalize` requires Node with ICU (official downloads include it).

---

### Timestamp resolution

- Filesystems differ: nanoseconds, milliseconds, 1s, 2s, even 24h atime on some FAT variants.
- Preserve resolution you receive; compare with the FS’s effective granularity where needed.

---

### Practical guidance

- Preserve exact filenames/timestamps from the FS.
- Use normalization and case-folding only for comparisons, not storage.
- If the FS lacks permissions/extended attributes, don’t expect round-trips to match on other platforms.
- Consider per-path behavior (different mounts under the same tree may differ).

---

> ⚠️ **Pitfalls**
>
> - **Inferring from OS**: Assuming macOS means case-insensitive, or Linux means full Unix perms — not always true.
> - **Lowest common denominator**: Forcing uppercase or single Unicode form corrupts data and causes collisions.
> - **Mutating data via normalization**: Using `normalize()` to store filenames — comparison only, never storage.
> - **Mismatched comparisons**: Case-insensitive compare on a case-sensitive FS (or vice versa) leads to bugs.
> - **Timestamp downsampling**: Truncating times globally can break sync/backup logic across FSes.
> - **Assuming uniform mounts**: Different volumes under one tree can have different behaviors.

---

### Interview-ready talking points

- Why inferring FS behavior from `process.platform` is unsafe; probing vs assuming.
- Superset approach: preserve case/Unicode/timestamps/permissions, downsample only when necessary.
- Unicode: NFC vs NFD, preservation vs insensitivity, proper use of `normalize()`.
- Case preservation vs sensitivity and their implications for rename detection and dedup.
- Timestamp resolution differences and safe comparison strategies.

---

### Quick checklist

- [ ] Are we preserving filenames and timestamps exactly as read?
- [ ] Are comparisons (case/Unicode/timestamps) adapted to the active filesystem?
- [ ] Do we avoid normalizing or uppercasing stored filenames?
- [ ] Have we considered per-mount differences within the working tree?
