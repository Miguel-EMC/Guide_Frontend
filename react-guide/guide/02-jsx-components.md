# 02 - JSX & Components

JSX lets you write HTML-like syntax inside JavaScript. Components are functions that return JSX.

## JSX Basics

```tsx
const title = 'Dashboard';

export function Header() {
  return (
    <header>
      <h1>{title}</h1>
      <p>{new Date().toLocaleDateString()}</p>
    </header>
  );
}
```

### Rules of JSX

- Must return a single root element (or Fragment)
- Use `{}` to embed JS expressions
- Use `className`, `htmlFor`, and camelCase props
- Component names must start with a capital letter

```tsx
export function Card() {
  return (
    <>
      <h2>Profile</h2>
      <p className="muted">Member</p>
    </>
  );
}
```

## Component Composition

```tsx
import type { ReactNode } from 'react';

function Button({ children }: { children: ReactNode }) {
  return <button className="btn">{children}</button>;
}

function Toolbar() {
  return (
    <div className="toolbar">
      <Button>Save</Button>
      <Button>Cancel</Button>
    </div>
  );
}
```

## Props and Children

```tsx
import type { ReactNode } from 'react';

function Panel({ title, children }: { title: string; children: ReactNode }) {
  return (
    <section>
      <h2>{title}</h2>
      <div>{children}</div>
    </section>
  );
}
```

## Next Steps

- [Props & State](./03-props-state.md)
- [Conditional & List Rendering](./04-rendering.md)

---

[Previous: Introduction](./01-introduction.md) | [Back to Index](./README.md) | [Next: Props & State](./03-props-state.md)
