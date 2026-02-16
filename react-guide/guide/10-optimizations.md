# 10 - useMemo, useCallback & Optimizations

Optimization tools help avoid unnecessary renders when used carefully.

## memo

```tsx
import { memo } from 'react';

const Card = memo(function Card({ title }: { title: string }) {
  return <h3>{title}</h3>;
});
```

## useMemo

```tsx
import { useMemo } from 'react';

function Stats({ items }: { items: number[] }) {
  const total = useMemo(() => items.reduce((a, b) => a + b, 0), [items]);
  return <p>Total: {total}</p>;
}
```

## useCallback

```tsx
import { useCallback } from 'react';

function List({ onSelect }: { onSelect: (id: string) => void }) {
  const handleSelect = useCallback((id: string) => onSelect(id), [onSelect]);
  return <button onClick={() => handleSelect('1')}>Select</button>;
}
```

## When to Optimize

- Component renders are expensive
- Props change frequently
- Lists are large and re-render often

## Next Steps

- [Context API in Depth](./11-context-api.md)
- [Portals & Error Boundaries](./12-portals-error-boundaries.md)

---

[Previous: useRef & Custom Hooks](./09-useref-custom-hooks.md) | [Back to Index](./README.md) | [Next: Context API in Depth](./11-context-api.md)
