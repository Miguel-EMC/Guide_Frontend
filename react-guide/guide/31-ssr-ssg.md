# 31 - SSR & SSG with Next.js and Remix

SSR renders HTML on the server, while SSG pre-renders pages at build time. React frameworks handle this for you.

## When to Use SSR or SSG

- SEO sensitive pages and fast first paint
- Large apps that benefit from streaming HTML
- Content heavy sites where data changes slowly

## React 19 Static Rendering APIs

```tsx
import { prerender } from 'react-dom/static';
import { App } from './App';

export async function renderPage() {
  const { prelude } = await prerender(<App />);
  return prelude;
}
```

## Framework Options

- Next.js: full stack, file based routing, server actions
- Remix: nested routes with loaders and actions
- React Router framework: data routers and mutations

## Tips

- Keep data fetching close to routes
- Avoid leaking secrets into the client bundle
- Use streaming and Suspense for fast TTFB

## Next Steps

- [Project: E-commerce Store](./32-e-commerce-store.md)

---

[Previous: Deployment Strategies](./30-deployment.md) | [Back to Index](./README.md) | [Next: Project: E-commerce Store](./32-e-commerce-store.md)
