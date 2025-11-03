# Reacting to Input with State

## Overview
React uses a declarative way to manipulate the UI. Instead of manipulating individual pieces of the UI directly, you describe the different states your component can be in, and switch between them in response to user input.

## What You Will Learn
- How declarative UI programming differs from imperative approaches
- Methods for enumerating different visual states a component can display
- Techniques for triggering state changes between visual states through code

---

## Declarative vs. Imperative UI Programming

### Imperative Programming

In **imperative programming**, you write explicit instructions for exactly how to manipulate the UI. Like giving turn-by-turn directions to a driver:

```jsx
// Imperative approach (without React)
form.disable();
submitButton.disable();
spinner.show();

fetch('/submit', { method: 'POST', body: data })
  .then(() => {
    spinner.hide();
    successMessage.show();
    form.hide();
  })
  .catch(error => {
    spinner.hide();
    errorMessage.show();
    errorMessage.textContent = error.message;
    form.enable();
  });
```

**Problems:**
- Must manually update every element
- Easy to forget steps or create inconsistent states
- Complexity grows exponentially with more UI states
- Hard to maintain and extend

### Declarative Programming

In **declarative programming**, you declare what you want to show for each state. React figures out how to update the UI. Like telling a taxi driver your destination (not every turn):

```jsx
// Declarative approach (with React)
function Form({ status }) {
  if (status === 'success') {
    return <h1>Thank you!</h1>;
  }

  return (
    <form>
      <textarea disabled={status === 'submitting'} />
      <button disabled={status === 'empty' || status === 'submitting'}>
        Submit
      </button>
      {status === 'error' && <p className="error">Error occurred!</p>}
    </form>
  );
}
```

**Benefits:**
- Describe UI for each state once
- React handles all DOM updates
- Easier to reason about
- Fewer bugs from forgotten updates
- Easier to extend with new states

---

## Thinking About UI Declaratively: A Five-Step Process

### Step 1: Identify All Visual States

List **all possible visual states** your component can display.

**Example: Form Component**
1. **Empty:** Submit button disabled
2. **Typing:** Submit button enabled
3. **Submitting:** Form completely disabled, spinner showing
4. **Success:** "Thank you" message shown, form hidden
5. **Error:** Same as Typing state, but with error message

**Tip:** For complex UIs, mock each state visually before adding logic. This helps you think through all scenarios.

```jsx
// Create a "living styleguide" to visualize all states
export default function FormStates() {
  return (
    <>
      <Form status="empty" />
      <Form status="typing" />
      <Form status="submitting" />
      <Form status="success" />
      <Form status="error" />
    </>
  );
}
```

---

### Step 2: Determine What Triggers State Changes

State changes come from two types of inputs:

**Human inputs:**
- Clicking a button
- Typing in a field
- Navigating a link

**Computer inputs:**
- Network response arriving
- Timeout completing
- Image loading

**Both require state variables to update the UI.**

**Example triggers:**
- Typing in textarea → Set state to "typing"
- Click Submit → Set state to "submitting"
- Network success → Set state to "success"
- Network failure → Set state to "error"

---

### Step 3: Represent State with `useState`

Start by identifying the **essential state** your component needs.

**Initial attempt (form example):**
```jsx
const [answer, setAnswer] = useState('');
const [error, setError] = useState(null);
const [status, setStatus] = useState('typing'); // 'typing', 'submitting', 'success'
```

This captures the minimum information needed to represent all visual states.

---

### Step 4: Remove Non-Essential State Variables

**Goal:** Eliminate redundant or contradictory state to prevent bugs.

Ask these questions about each state variable:

#### Question 1: Does this state cause a paradox?

**❌ Bad: Contradictory states possible**
```jsx
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// Paradox: Can both be true? Can both be false?
```

**✅ Good: Use a single variable with explicit values**
```jsx
const [status, setStatus] = useState('typing');
// Possible values: 'typing' | 'sending' | 'sent'
// Only one value at a time—no paradoxes!
```

#### Question 2: Is the same information available in another state variable?

**❌ Bad: Redundant state**
```jsx
const [answer, setAnswer] = useState('');
const [isEmpty, setIsEmpty] = useState(true);

// isEmpty duplicates information from answer
```

**✅ Good: Derive it during render**
```jsx
const [answer, setAnswer] = useState('');
const isEmpty = answer.length === 0; // Compute, don't store
```

#### Question 3: Can you get this from the inverse of another variable?

**❌ Bad: Mirror state**
```jsx
const [isTyping, setIsTyping] = useState(true);
const [isSubmitting, setIsSubmitting] = useState(false);

// isSubmitting = !isTyping (inverse relationship)
```

**✅ Good: Use one, derive the other**
```jsx
const [status, setStatus] = useState('typing');
const isSubmitting = status === 'submitting';
const isTyping = status === 'typing';
```

> **⚠️ PITFALL: STATE PARADOXES**
>
> **Avoid "impossible states" with multiple booleans:**
>
> ```jsx
> // ❌ Can create invalid combinations:
> const [isEmpty, setIsEmpty] = useState(true);
> const [isTyping, setIsTyping] = useState(false);
> const [isSubmitting, setIsSubmitting] = useState(false);
> const [isSuccess, setIsSuccess] = useState(false);
> const [isError, setIsError] = useState(false);
>
> // What if isSuccess and isError are both true? 🤔
> // What if isTyping and isSubmitting are both true? 🤔
> ```
>
> **✅ Use a single status variable:**
> ```jsx
> const [status, setStatus] = useState('typing');
> // Only one status at a time—impossible states eliminated!
> ```

---

### Step 5: Connect Event Handlers

Wire up event handlers to update state based on user actions:

```jsx
function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>That's right!</h1>;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea
        value={answer}
        onChange={handleTextareaChange}
        disabled={status === 'submitting'}
      />
      <button disabled={answer.length === 0 || status === 'submitting'}>
        Submit
      </button>
      {error && <p className="error">{error.message}</p>}
    </form>
  );
}
```

---

## Complete Example: Refactoring State

### ❌ Before: Too Many State Variables

```jsx
function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [isEmpty, setIsEmpty] = useState(true);
  const [isTyping, setIsTyping] = useState(false);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isSuccess, setIsSuccess] = useState(false);
  const [isError, setIsError] = useState(false);

  // Easy to create contradictory states!
  // Hard to keep all variables in sync
}
```

### ✅ After: Essential State Only

```jsx
function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing'); // 'typing' | 'submitting' | 'success'

  // Derive other values:
  const isEmpty = answer.length === 0;
  const isTyping = status === 'typing';
  const isSubmitting = status === 'submitting';
  const isSuccess = status === 'success';
  const isError = error !== null && status === 'typing';
}
```

---

## Benefits of Declarative UI

### 1. Easier to Maintain
Adding new states doesn't require updating scattered DOM manipulation code—just add a new condition.

### 2. Prevents Bugs
Impossible to create inconsistent UI states when you describe what to show rather than how to show it.

### 3. Easier to Extend
Adding features (like a loading spinner or retry button) is straightforward—add the state and describe what to render.

### 4. Better Collaboration
Designers and developers can work with visual states without understanding implementation details.

---

## Practical Examples

### Example 1: Traffic Light
```jsx
function TrafficLight() {
  const [light, setLight] = useState('red');

  return (
    <div>
      <div className={light === 'red' ? 'active' : ''}>Red</div>
      <div className={light === 'yellow' ? 'active' : ''}>Yellow</div>
      <div className={light === 'green' ? 'active' : ''}>Green</div>
      <button onClick={() => {
        if (light === 'red') setLight('green');
        if (light === 'green') setLight('yellow');
        if (light === 'yellow') setLight('red');
      }}>
        Next
      </button>
    </div>
  );
}
```

### Example 2: Login Form
```jsx
function LoginForm() {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'error' | 'success'
  const [error, setError] = useState(null);

  const canSubmit = username.length > 0 && password.length > 0 && status !== 'loading';

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('loading');
    setError(null);

    try {
      await login(username, password);
      setStatus('success');
    } catch (err) {
      setStatus('error');
      setError(err.message);
    }
  }

  if (status === 'success') {
    return <h1>Welcome back!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={username}
        onChange={e => setUsername(e.target.value)}
        disabled={status === 'loading'}
      />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
        disabled={status === 'loading'}
      />
      <button disabled={!canSubmit}>
        {status === 'loading' ? 'Logging in...' : 'Login'}
      </button>
      {status === 'error' && <p className="error">{error}</p>}
    </form>
  );
}
```

---

## Interview Questions to Consider

1. **Q: What's the difference between imperative and declarative UI programming?**
   - A: Imperative programming requires explicit DOM manipulation instructions. Declarative programming describes what the UI should look like for each state, and React handles the updates.

2. **Q: What are the five steps for thinking about UI declaratively?**
   - A: (1) Identify all visual states, (2) Determine what triggers state changes, (3) Represent state with useState, (4) Remove non-essential state, (5) Connect event handlers.

3. **Q: Why should you avoid multiple boolean state variables?**
   - A: Multiple booleans can create "impossible states" (contradictory combinations). A single status variable with explicit values prevents this.

4. **Q: How do you know if a state variable is necessary?**
   - A: Ask: Does it cause paradoxes? Is it already available elsewhere? Can you derive it from another variable? If yes to any, it's probably not essential.

5. **Q: What's a "state paradox"?**
   - A: When multiple state variables can be in contradictory combinations, like `isTyping` and `isSubmitting` both being true simultaneously.

6. **Q: Should you store derived values in state?**
   - A: No! Compute them during render instead. For example, don't store `isEmpty` if you can compute `answer.length === 0`.

7. **Q: Why is declarative UI programming easier to maintain?**
   - A: You describe what to show for each state in one place, rather than scattering DOM updates throughout your code. Adding states doesn't require hunting down all the places you manipulate the DOM.

8. **Q: What's the benefit of creating a "living styleguide" showing all states?**
   - A: It helps you think through all possible UI states before writing logic, catches edge cases early, and serves as documentation.

---

## Key Takeaways

- **Declarative UI:** Describe what to show for each state; React handles how to update the DOM
- **Imperative UI:** Manually manipulate each DOM element (error-prone and hard to maintain)
- **Five-step process:** Identify states → Determine triggers → Add useState → Remove non-essential state → Connect handlers
- **Eliminate impossible states** by using a single status variable instead of multiple booleans
- **Don't store derived values** in state—compute them during render
- **Visual states first:** Enumerate all possible UI states before adding interactivity
- State should represent **minimal, non-redundant information**
- Ask three questions to identify non-essential state: Paradoxes? Already available? Inverse of another?
- Declarative programming reduces bugs and makes code easier to extend
- Think of UI as state machines with explicit transitions between states

