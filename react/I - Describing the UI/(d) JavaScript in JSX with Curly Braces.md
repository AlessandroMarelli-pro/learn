# JavaScript in JSX with Curly Braces

## Overview
JSX allows you to write markup in JavaScript, but you often need to add dynamic content or logic. Curly braces are the gateway that lets you "escape back" into JavaScript from within JSX markup.

## What You Will Learn
- Passing strings using quotes in JSX attributes
- Referencing JavaScript variables within JSX markup using curly braces
- Calling JavaScript functions inside JSX with curly braces
- Using JavaScript objects within JSX

---

## Strings with Quotes vs. Dynamic Values

### Static Strings: Use Quotes
When passing a **static string** attribute, use single or double quotes:

```jsx
<img
  className="avatar"
  src="https://example.com/photo.jpg"
  alt="Profile photo"
/>
```

### Dynamic Values: Use Curly Braces
When you need to use a **JavaScript variable or expression**, replace quotes with curly braces:

```jsx
const avatar = "https://example.com/photo.jpg";
const description = "Profile photo";

<img
  className="avatar"
  src={avatar}
  alt={description}
/>
```

**Key Distinction:** `src="{avatar}"` passes the literal string `"{avatar}"`, while `src={avatar}` passes the variable's value.

---

## Curly Braces: The JavaScript Gateway

**"Curly braces let you bring JavaScript logic and variables into your markup."**

Any valid JavaScript expression works inside curly braces:
- Variables: `{name}`
- Function calls: `{formatDate(today)}`
- Mathematical operations: `{price * 1.2}`
- String concatenation: `{firstName + ' ' + lastName}`
- Ternary expressions: `{isActive ? 'active' : 'inactive'}`

---

## Where You Can Use Curly Braces

Curly braces work in **two specific places** within JSX:

### 1. As Text Content (Inside Tags)
```jsx
<h1>{name}'s To Do List</h1>
```

This renders the value of `name` followed by "'s To Do List".

### 2. As Attribute Values (After `=`)
```jsx
<img src={avatar} alt={description} />
```

This reads the `avatar` and `description` variables.

> **⚠️ PITFALL**
>
> `src={avatar}` passes the variable value, but `src="{avatar}"` passes the literal string `"{avatar}"`. Don't wrap curly braces in quotes when you want dynamic values!

---

## Using JavaScript Functions

You can call functions directly inside JSX:

```jsx
function formatDate(date) {
  return date.toLocaleDateString();
}

function TodoList() {
  const today = new Date();

  return (
    <h1>To Do List for {formatDate(today)}</h1>
  );
}
```

---

## Double Curly Braces: Objects in JSX

### Understanding "Double Curlies"

When you see `{{ }}` in JSX, it's **not special syntax**—it's simply a JavaScript object inside JSX curly braces.

**Breaking it down:**
- Outer `{ }`: JSX expression delimiter
- Inner `{ }`: JavaScript object literal

### Inline Styles with Objects

React inline styles are written as JavaScript objects:

```jsx
<div style={{
  backgroundColor: 'black',
  color: 'pink',
  padding: '10px'
}}>
  Styled content
</div>
```

**Equivalent to:**
```jsx
const styles = {
  backgroundColor: 'black',
  color: 'pink',
  padding: '10px'
};

<div style={styles}>
  Styled content
</div>
```

> **⚠️ PITFALL**
>
> Inline `style` properties are written in **camelCase**. HTML's `background-color` becomes `backgroundColor`, `font-size` becomes `fontSize`, etc. This follows JavaScript property naming conventions.

---

## CSS Properties in camelCase

When using inline styles, CSS properties follow JavaScript naming rules:

| HTML/CSS | React JSX |
|----------|-----------|
| `background-color` | `backgroundColor` |
| `font-size` | `fontSize` |
| `border-radius` | `borderRadius` |
| `margin-top` | `marginTop` |
| `z-index` | `zIndex` |

**Example:**
```jsx
<div style={{
  fontSize: '16px',
  marginTop: '20px',
  borderRadius: '8px'
}}>
  Content
</div>
```

---

## Practical Examples

### Dynamic Image URL and Alt Text
```jsx
const user = {
  name: 'Alex Johnson',
  imageUrl: 'https://example.com/alex.jpg',
  imageSize: 90
};

<img
  src={user.imageUrl}
  alt={'Photo of ' + user.name}
  style={{
    width: user.imageSize,
    height: user.imageSize
  }}
/>
```

### Nested Object Property Access
```jsx
const person = {
  profile: {
    name: 'Maria Garcia',
    theme: {
      backgroundColor: 'black',
      color: 'pink'
    }
  }
};

<div style={person.profile.theme}>
  <h1>{person.profile.name}</h1>
</div>
```

### String Concatenation for URLs
```jsx
const baseUrl = 'https://i.imgur.com/';
const imageId = '7vQD0fP';
const imageSize = 's';

<img
  src={baseUrl + imageId + imageSize + '.jpg'}
  alt="Sculpture"
/>
```

---

## What You Cannot Render

> **⚠️ PITFALL**
>
> React components **cannot render raw objects** as children. This will cause an error:
> ```jsx
> const person = { name: 'Maria' };
> return <h1>{person}</h1>; // ❌ Error!
> ```
>
> You must access specific properties:
> ```jsx
> return <h1>{person.name}</h1>; // ✅ Works!
> ```

Objects are only valid for:
- Style attributes: `style={{ color: 'red' }}`
- Props and state values (passed to components)

---

## Interview Questions to Consider

1. **Q: What's the difference between `src={avatar}` and `src="{avatar}"`?**
   - A: `src={avatar}` passes the value of the `avatar` variable, while `src="{avatar}"` passes the literal string `"{avatar}"`.

2. **Q: Where can you use curly braces in JSX?**
   - A: In two places: (1) as text content directly inside tags, and (2) as attribute values immediately after the `=` sign.

3. **Q: What does `{{ backgroundColor: 'red' }}` represent in JSX?**
   - A: It's a JavaScript object literal (inner braces) wrapped in JSX expression delimiters (outer braces). The outer braces tell JSX "this is JavaScript", and the inner braces define an object.

4. **Q: Why do CSS properties use camelCase in React inline styles?**
   - A: Because inline styles are JavaScript objects, and object property names follow JavaScript conventions. Dashes aren't valid in JavaScript identifiers without quotes.

5. **Q: Can you render an object directly in JSX like `<h1>{person}</h1>`?**
   - A: No, this will throw an error. You must access specific primitive properties like `{person.name}` or convert the object to a string/number.

6. **Q: What JavaScript expressions can you use inside curly braces?**
   - A: Any valid JavaScript expression: variables, function calls, mathematical operations, ternary operators, string concatenation, property access, etc.

---

## Key Takeaways

- Use **quotes for static strings**, **curly braces for dynamic JavaScript**
- Curly braces work in text content and attribute values
- Any valid JavaScript expression can go inside `{ }`
- Double curlies `{{ }}` represent an object inside JSX braces
- CSS properties in inline styles use camelCase
- Objects cannot be rendered directly as children—access their properties instead
- Curly braces are your gateway between JSX markup and JavaScript logic
