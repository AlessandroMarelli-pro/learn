### Template Literals — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals

### 1) What they are

- Backtick‑delimited strings allowing multi‑line text, interpolation with `${expr}`, and tagged templates for custom processing.

```js
const a = 5,
  b = 10;
const msg = `Fifteen is ${a + b} and\nnot ${2 * a + b}.`;
```

### 2) Multi‑line and escaping

- Newlines in the source are preserved. Escape backticks with ` \``. Escape  `${`as`\${` to prevent interpolation.

```js
const multi = `line 1
line 2`;
`\`` === '`'; // true
```

### 3) Expression interpolation

- `${expr}` evaluates any JS expression and coerces the result to string (default tag). Prefer templates over `+` concatenation for readability.

```js
const user = { name: 'Ava', score: 42 };
const line = `${user.name} — ${user.score}`;
```

### 4) Nesting templates

- You can nest for readability and conditional fragments.

```js
const cls = `header ${
  isLarge() ? '' : `icon-${item.collapsed ? 'expander' : 'collapser'}`
}`;
```

### 5) Tagged templates (custom processing)

- Prefix with a function to customize how literals and substitutions are combined. First arg is an array of string parts; rest are values.

```js
function safe(strings, ...values) {
  const esc = (s) => String(s).replaceAll('<', '&lt;');
  return strings
    .map((s, i) => s + (i < values.length ? esc(values[i]) : ''))
    .join('');
}
const out = safe`<div>${user.name}</div>`; // escapes `<`
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Invalid escape sequences in untagged templates are syntax errors; with a tag, they are passed through to your function.
> - Interpolation uses string coercion; objects may print as `[object Object]` unless you format them.
> - Backticks inside templates require escaping unless placed within an `${expr}`.
> - Beware injection: use tagged templates or explicit escaping for HTML/SQL contexts.
> - Template literals don’t trim whitespace automatically; trailing spaces/newlines are significant.

### Interview‑Style Questions (Practice)

- Compare `"a" + x + "b"` vs `` `a${x}b` `` in coercion and readability.
- How do you safely include a backtick and `${` in a template literal?
- Demonstrate a tagged template that escapes HTML; explain arguments passed to the tag.
- Show nested templates to build a conditional class string.
- What changes when a tag is present regarding escape sequence handling?

### References

- MDN — Template literals: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
