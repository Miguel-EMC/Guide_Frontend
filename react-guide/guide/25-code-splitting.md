# 25 - Code Splitting & Lazy Loading

Code splitting reduces initial bundle size by loading code only when needed.

## Component Level Splitting

```tsx
import { Suspense } from 'react';

const Admin = React.lazy(() => import('./Admin'));

export function App() {
  return (
    <Suspense fallback={<p>Loading admin...</p>}>
      <Admin />
    </Suspense>
  );
}
```

## Route Level Splitting

```tsx
import { createBrowserRouter } from 'react-router';

const router = createBrowserRouter([
  {
    path: '/settings',
    lazy: async () => {
      const mod = await import('./routes/Settings');
      return { Component: mod.Settings };
    }
  }
]);
```

## Tips

- Split by routes first for the biggest impact
- Keep loading UI consistent across pages
- Preload critical chunks on hover or idle

## Next Steps

- [Performance Best Practices](./26-performance.md)
- [Unit Testing with RTL/Vitest](./27-testing.md)

---

[Previous: Advanced Data Fetching](./24-advanced-data-fetching.md) | [Back to Index](./README.md) | [Next: Performance Best Practices](./26-performance.md)
