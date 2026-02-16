# 13 - Concurrent React (Suspense, useTransition)

Concurrent features help keep the UI responsive during expensive updates and data loading.

## Suspense and Lazy Loading

```tsx
import { Suspense } from 'react';

const Settings = React.lazy(() => import('./Settings'));

export function App() {
  return (
    <Suspense fallback={<p>Loading settings...</p>}>
      <Settings />
    </Suspense>
  );
}
```

## useTransition for Low Priority Updates

```tsx
import { useState, useTransition } from 'react';

export function Search() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  return (
    <div>
      <input
        value={query}
        onChange={(e) => {
          const next = e.target.value;
          startTransition(() => setQuery(next));
        }}
      />
      {isPending && <span>Updating...</span>}
    </div>
  );
}
```

## useDeferredValue for Large Lists

```tsx
import { useDeferredValue, useMemo } from 'react';

export function List({ items }: { items: string[] }) {
  const deferred = useDeferredValue(items);
  const list = useMemo(() => deferred.slice(0, 200), [deferred]);
  return <ul>{list.map((item) => <li key={item}>{item}</li>)}</ul>;
}
```

## use() with Suspense

```tsx
import { Suspense, use } from 'react';

const userPromise = fetch('/api/user').then((r) => r.json());

function User() {
  const user = use(userPromise);
  return <p>Hi {user.name}</p>;
}

export function App() {
  return (
    <Suspense fallback={<p>Loading user...</p>}>
      <User />
    </Suspense>
  );
}
```

## Next Steps

- [TypeScript with React](./14-typescript.md)
- [State Management](./15-state-management.md)

---

[Previous: Portals & Error Boundaries](./12-portals-error-boundaries.md) | [Back to Index](./README.md) | [Next: TypeScript with React](./14-typescript.md)
