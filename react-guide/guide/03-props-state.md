# 03 - Props & State

Props let you pass data into components. State lets components manage local data.

## Props

```tsx
function UserCard({ name, role }: { name: string; role: string }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{role}</p>
    </div>
  );
}
```

### Props Tips

- Props are read-only. Never mutate them directly.
- Keep props small and focused on what the component needs.

## State with useState

```tsx
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount((c) => c + 1)}>+</button>
    </div>
  );
}
```

### State Updates

- Use functional updates for derived state.
- Avoid duplicating derived values in state.
- Never mutate objects or arrays in place.

```tsx
const [items, setItems] = useState<string[]>([]);
const total = items.length; // derived

setItems((prev) => [...prev, 'new']);
```

## Lifting State Up

```tsx
function Parent() {
  const [query, setQuery] = useState('');
  return <Search value={query} onChange={setQuery} />;
}

function Search({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

## Next Steps

- [Conditional & List Rendering](./04-rendering.md)
- [Handling Events](./05-events.md)

---

[Previous: JSX & Components](./02-jsx-components.md) | [Back to Index](./README.md) | [Next: Conditional & List Rendering](./04-rendering.md)
