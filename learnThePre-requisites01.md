# Pre-requisites for React Native

Before diving into React Native, it's important to be comfortable with **JavaScript basics**, **CSS basics**, and **React basics** (JSX, Components, State, Props). These form the foundation for everything in React Native.

---

## 📜 1. JavaScript Basics

React Native is built on top of JavaScript, so understanding these core concepts is essential.

---

### Variables (`let`, `const`, `var`)

```javascript
let name = "Manisha";       // can be reassigned
const age = 22;             // cannot be reassigned
var city = "Jaipur";        // old way, avoid using
```

---

### Functions

```javascript
// Normal function
function greet(name) {
    return `Hello, ${name}`;
}

// Arrow function (commonly used in React)
const greet = (name) => {
    return `Hello, ${name}`;
};

console.log(greet("Manisha")); // Hello, Manisha
```

---

### Arrays & Array Methods

```javascript
const numbers = [1, 2, 3, 4, 5];

// map() - transforms each element
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// filter() - keeps elements that match condition
const evenNumbers = numbers.filter(num => num % 2 === 0);
console.log(evenNumbers); // [2, 4]
```

---

### Objects & Destructuring

```javascript
const user = {
    name: "Manisha",
    age: 22,
    city: "Jaipur"
};

// Destructuring
const { name, age } = user;
console.log(name, age); // Manisha 22
```

---

### Spread Operator (`...`)

```javascript
const oldData = { id: 1, name: "Manisha" };

// Creates a new object, keeping old data and updating 'name'
const newData = { ...oldData, name: "Kumari" };
console.log(newData); // { id: 1, name: "Kumari" }
```

> 💡 This is heavily used in React state updates (e.g., `setFormData({...formData, name: value})`)

---

### Ternary Operator

```javascript
const isLoggedIn = true;

const message = isLoggedIn ? "Welcome back!" : "Please log in";
console.log(message); // Welcome back!
```

---

### Async/Await (for API calls)

```javascript
const fetchData = async () => {
    try {
        const response = await fetch('https://api.example.com/data');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.log("Error:", error);
    }
};
```

---

## 🎨 2. CSS Basics

CSS concepts (especially **Flexbox**) carry directly into React Native styling.

---

### Box Model

```css
.box {
    width: 100px;
    height: 100px;
    padding: 10px;     /* space inside the box */
    margin: 20px;       /* space outside the box */
    border: 2px solid black;
}
```

---

### Flexbox (Most Important for React Native)

```css
.container {
    display: flex;
    flex-direction: row;        /* row or column */
    justify-content: center;    /* aligns items on main axis */
    align-items: center;        /* aligns items on cross axis */
}
```

| Property | Purpose |
|---|---|
| `flex-direction` | Layout direction: row or column |
| `justify-content` | Aligns items along the main axis |
| `align-items` | Aligns items along the cross axis |

> 💡 React Native uses the **same Flexbox concepts** (camelCase, default `flexDirection: 'column'`)

---

### Colors & Units

```css
.text {
    color: #333333;
    background-color: rgba(0, 0, 0, 0.5);
    font-size: 18px;
}
```

---

## ⚛️ 3. React Basics

React Native is built using **React**, so these four concepts are the foundation of every screen you build.

---

### JSX (JavaScript Syntax Extension)

JSX lets you write HTML-like syntax inside JavaScript.

```jsx
const element = <h1>Hello, World!</h1>;
```

This compiles to:

```javascript
React.createElement('h1', null, 'Hello, World!');
```

**Rules of JSX:**
- Must return a single parent element
- Use `{}` to insert JavaScript expressions

```jsx
const name = "Manisha";

const Greeting = () => {
    return (
        <div>
            <h1>Hello, {name}!</h1>
            <p>{1 + 1}</p>  {/* renders 2 */}
        </div>
    );
};
```

---

### Components

Components are **reusable building blocks** of a UI.

```jsx
// Functional Component
const Welcome = () => {
    return <h1>Welcome to React!</h1>;
};

// Using the component
const App = () => {
    return (
        <div>
            <Welcome />
            <Welcome />
        </div>
    );
};
```

---

### Props (Passing Data to Components)

Props let you pass data **from parent to child** components.

```jsx
// Child component receives props
const Welcome = (props) => {
    return <h1>Hello, {props.name}!</h1>;
};

// Parent passes data via props
const App = () => {
    return (
        <div>
            <Welcome name="Manisha" />
            <Welcome name="Ali" />
        </div>
    );
};
```

**With Destructuring (cleaner):**

```jsx
const Welcome = ({ name, age }) => {
    return <h1>{name} is {age} years old</h1>;
};

const App = () => {
    return <Welcome name="Manisha" age={22} />;
};
```

> ⚠️ Props are **read-only** — a child component cannot modify the props it receives.

---

### State (`useState`)

State holds data that **changes over time** and causes the component to re-render.

```jsx
import React, { useState } from 'react';

const Counter = () => {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>
                Increment
            </button>
        </div>
    );
};
```

**Object state (always spread old data when updating):**

```jsx
const [formData, setFormData] = useState({ name: '', email: '' });

const handleChange = (field, value) => {
    setFormData({ ...formData, [field]: value });
};
```

---

### Props vs State

| | Props | State |
|---|---|---|
| **Definition** | Data passed from parent to child | Data managed within the component |
| **Mutability** | Read-only | Can be updated using setter function |
| **Who controls it** | Parent component | The component itself |
| **Triggers re-render?** | Yes, when parent passes new props | Yes, when state is updated |

---

## Conclusion

- **JavaScript basics** (variables, functions, arrays, objects, spread, async/await) form the logic layer
- **CSS basics** (especially Flexbox) directly translate into React Native styling
- **React basics** (JSX, Components, Props, State) are the building blocks of every React Native screen

Mastering these fundamentals makes React Native much easier to learn, since React Native is essentially **React + Native Components** instead of HTML elements.


# Important React Concepts & Hooks

Before diving into React Native, these React concepts and hooks are essential to understand deeply.

---

## ⚛️ 1. Other Important React Concepts

---

### Conditional Rendering

Render different UI based on a condition.

```jsx
const App = () => {
    const isLoggedIn = true;

    return (
        <div>
            {isLoggedIn ? (
                <h1>Welcome Back!</h1>
            ) : (
                <h1>Please Log In</h1>
            )}
        </div>
    );
};
```

**Using `&&` (short-circuit):**

```jsx
const App = () => {
    const hasError = true;

    return (
        <div>
            {hasError && <p>Something went wrong!</p>}
        </div>
    );
};
```

---

### Lists & Keys

Render a list of items using `.map()`. Always provide a unique `key` prop.

```jsx
const fruits = ['Apple', 'Banana', 'Mango'];

const FruitList = () => {
    return (
        <ul>
            {fruits.map((fruit, index) => (
                <li key={index}>{fruit}</li>
            ))}
        </ul>
    );
};
```

> ⚠️ Always use a unique `key` — React uses it to track which items changed.

---

### Event Handling

```jsx
const Button = () => {
    const handleClick = () => {
        alert("Button clicked!");
    };

    return <button onClick={handleClick}>Click Me</button>;
};
```

**Passing arguments to event handler:**

```jsx
const handleClick = (name) => {
    alert(`Hello, ${name}`);
};

<button onClick={() => handleClick("Manisha")}>Click</button>
```

---

### Forms & Controlled Components

In React, form inputs are controlled by state.

```jsx
import React, { useState } from 'react';

const LoginForm = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        console.log(email, password);
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="email"
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                placeholder="Enter Email"
            />
            <input
                type="password"
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                placeholder="Enter Password"
            />
            <button type="submit">Login</button>
        </form>
    );
};
```

---

### Component Lifecycle (Simplified)

Every React component goes through 3 phases:

| Phase | Description |
|---|---|
| **Mounting** | Component is created and added to the DOM |
| **Updating** | Component re-renders when state or props change |
| **Unmounting** | Component is removed from the DOM |

> 💡 In functional components, all lifecycle phases are handled using **`useEffect`** hook.

---

### Lifting State Up

When two sibling components need to share data, move the state to their **common parent**.

```jsx
const Parent = () => {
    const [count, setCount] = useState(0);

    return (
        <div>
            <ChildA count={count} />
            <ChildB setCount={setCount} />
        </div>
    );
};

const ChildA = ({ count }) => <p>Count: {count}</p>;

const ChildB = ({ setCount }) => (
    <button onClick={() => setCount(prev => prev + 1)}>
        Increment
    </button>
);
```

---

### Importing & Exporting

```jsx
// Named Export
export const greet = () => "Hello!";
import { greet } from './utils';

// Default Export (only one per file)
export default function App() {}
import App from './App';
```

---

## 🪝 2. React Hooks

Hooks let you use **state and lifecycle features** in functional components. They always start with `use`.

> ⚠️ Rules of Hooks:
> - Only call hooks at the **top level** (not inside loops or conditions)
> - Only call hooks inside **React functional components**

---

### `useState` — Manage State

```jsx
import React, { useState } from 'react';

const Counter = () => {
    const [count, setCount] = useState(0);

    return (
        <div>
            <p>Count: {count}</p>
            <button onClick={() => setCount(count + 1)}>+</button>
            <button onClick={() => setCount(count - 1)}>-</button>
            <button onClick={() => setCount(0)}>Reset</button>
        </div>
    );
};
```

**Updating object state:**

```jsx
const [user, setUser] = useState({ name: '', age: 0 });

// Always spread old state first
setUser({ ...user, name: "Manisha" });
```

---

### `useEffect` — Side Effects & Lifecycle

Runs **after render**. Used for API calls, subscriptions, timers, etc.

```jsx
import React, { useState, useEffect } from 'react';

const App = () => {
    const [data, setData] = useState([]);

    // Runs once when component mounts (like componentDidMount)
    useEffect(() => {
        fetch('https://jsonplaceholder.typicode.com/posts')
            .then(res => res.json())
            .then(json => setData(json));
    }, []); // empty array = run only once

    return (
        <ul>
            {data.map(post => (
                <li key={post.id}>{post.title}</li>
            ))}
        </ul>
    );
};
```

**Dependency Array:**

```jsx
// Runs every render (no array)
useEffect(() => { console.log("Every render"); });

// Runs only once on mount (empty array)
useEffect(() => { console.log("Mounted"); }, []);

// Runs when 'count' changes
useEffect(() => { console.log("Count changed:", count); }, [count]);
```

**Cleanup function (like componentWillUnmount):**

```jsx
useEffect(() => {
    const timer = setInterval(() => {
        console.log("Tick");
    }, 1000);

    // Cleanup when component unmounts
    return () => clearInterval(timer);
}, []);
```

---

### `useRef` — Access DOM / Persist Values

`useRef` holds a value that **does not trigger re-render** when changed.

```jsx
import React, { useRef } from 'react';

const InputFocus = () => {
    const inputRef = useRef(null);

    const handleClick = () => {
        inputRef.current.focus(); // directly access the DOM element
    };

    return (
        <div>
            <input ref={inputRef} type="text" placeholder="Click button to focus" />
            <button onClick={handleClick}>Focus Input</button>
        </div>
    );
};
```

**Store previous value (no re-render):**

```jsx
const countRef = useRef(0);
countRef.current += 1; // updates value without re-rendering
```

---

### `useContext` — Global State (Avoid Prop Drilling)

Share data across many components without passing props at every level.

```jsx
import React, { createContext, useContext, useState } from 'react';

// 1. Create context
const UserContext = createContext();

// 2. Provide context at top level
const App = () => {
    const [user, setUser] = useState({ name: "Manisha" });

    return (
        <UserContext.Provider value={user}>
            <Dashboard />
        </UserContext.Provider>
    );
};

// 3. Consume context anywhere in the tree
const Dashboard = () => {
    const user = useContext(UserContext);
    return <h1>Welcome, {user.name}!</h1>;
};
```

> 💡 `useContext` solves **prop drilling** — passing props through many layers of components.

---

### `useReducer` — Complex State Management

Alternative to `useState` for complex state logic (like Redux, but built-in).

```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

const reducer = (state, action) => {
    switch (action.type) {
        case 'INCREMENT': return { count: state.count + 1 };
        case 'DECREMENT': return { count: state.count - 1 };
        case 'RESET':     return { count: 0 };
        default:          return state;
    }
};

const Counter = () => {
    const [state, dispatch] = useReducer(reducer, initialState);

    return (
        <div>
            <p>Count: {state.count}</p>
            <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
            <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
            <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
        </div>
    );
};
```

---

### `useMemo` — Optimize Performance (Memoize Values)

Caches the result of a calculation so it only re-runs when dependencies change.

```jsx
import React, { useState, useMemo } from 'react';

const App = () => {
    const [count, setCount] = useState(0);
    const [input, setInput] = useState('');

    // Only recalculates when 'count' changes, not when 'input' changes
    const expensiveResult = useMemo(() => {
        console.log("Calculating...");
        return count * 100;
    }, [count]);

    return (
        <div>
            <p>Result: {expensiveResult}</p>
            <button onClick={() => setCount(count + 1)}>Count: {count}</button>
            <input value={input} onChange={e => setInput(e.target.value)} />
        </div>
    );
};
```

---

### `useCallback` — Optimize Performance (Memoize Functions)

Caches a function so it's not recreated on every render.

```jsx
import React, { useState, useCallback } from 'react';

const App = () => {
    const [count, setCount] = useState(0);

    // Function is only recreated when 'count' changes
    const handleClick = useCallback(() => {
        console.log("Count is:", count);
    }, [count]);

    return (
        <div>
            <button onClick={handleClick}>Log Count</button>
            <button onClick={() => setCount(count + 1)}>Increment</button>
        </div>
    );
};
```

| Hook | `useMemo` | `useCallback` |
|---|---|---|
| Caches | A **value** | A **function** |
| Use when | Expensive calculations | Passing callbacks to child components |

---

### Custom Hook — Reusable Logic

You can create your own hook to reuse stateful logic across components.

```jsx
import { useState, useEffect } from 'react';

// Custom hook to fetch data
const useFetch = (url) => {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        fetch(url)
            .then(res => res.json())
            .then(json => {
                setData(json);
                setLoading(false);
            });
    }, [url]);

    return { data, loading };
};

// Using the custom hook
const App = () => {
    const { data, loading } = useFetch('https://jsonplaceholder.typicode.com/posts');

    if (loading) return <p>Loading...</p>;

    return (
        <ul>
            {data.map(post => (
                <li key={post.id}>{post.title}</li>
            ))}
        </ul>
    );
};
```

---

## 📋 Hooks Quick Reference

| Hook | Purpose |
|---|---|
| `useState` | Manage local state |
| `useEffect` | Handle side effects & lifecycle |
| `useRef` | Access DOM elements, persist values without re-render |
| `useContext` | Access global state, avoid prop drilling |
| `useReducer` | Manage complex state logic |
| `useMemo` | Memoize expensive computed values |
| `useCallback` | Memoize functions to avoid unnecessary re-creation |
| **Custom Hook** | Extract and reuse stateful logic across components |

---

## Conclusion

- **Conditional rendering, lists, events, and forms** are the everyday building blocks of any React app
- **Hooks** replace the need for class components entirely
- `useState` and `useEffect` are the **most commonly used** hooks
- `useContext` and `useReducer` are great for **state management**
- `useMemo` and `useCallback` are used for **performance optimization**
- **Custom hooks** keep your code clean and reusable

Mastering these concepts makes building complex React Native apps much easier.

---

*Notes on important React concepts and hooks — prerequisites for React Native.*



---


*Notes on JavaScript, CSS, and React pre-requisites for learning React Native.*