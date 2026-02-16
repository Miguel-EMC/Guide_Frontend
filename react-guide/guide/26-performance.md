# 26 - Performance Best Practices

Performance is about fast interactions and predictable rendering.

## Measure First

- Use React DevTools Profiler
- Measure real user metrics in production
- Optimize the slowest interactions first

## Rendering Optimizations

```tsx
const Row = React.memo(function Row({ item }: { item: string }) {
  return <li>{item}</li>;
});
```

```tsx
import { useMemo } from 'react';

function Stats({ items }: { items: number[] }) {
  const total = useMemo(() => items.reduce((a, b) => a + b, 0), [items]);
  return <p>Total: {total}</p>;
}
```

## List Virtualization

```tsx
import { FixedSizeList as List } from 'react-window';

export function VirtualList({ items }: { items: string[] }) {
  return (
    <List height={300} itemCount={items.length} itemSize={32} width="100%">
      {({ index, style }) => <div style={style}>{items[index]}</div>}
    </List>
  );
}
```

## React Compiler

React Compiler can automatically optimize rendering and reduce the need for manual memoization. Adopt it when your build toolchain supports it and validate with profiling.

## Tips

- Keep state close to where it is used
- Avoid recreating objects in render when possible
- Use Suspense and transitions to keep input responsive

## Next Steps

- [Unit Testing with RTL/Vitest](./27-testing.md)
- [End-to-End Testing](./28-e2e-testing.md)

---

[Previous: Code Splitting](./25-code-splitting.md) | [Back to Index](./README.md) | [Next: Unit Testing with RTL/Vitest](./27-testing.md)
