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

---

*Notes on JavaScript, CSS, and React pre-requisites for learning React Native.*