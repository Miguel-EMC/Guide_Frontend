# 32 - Project: E-commerce Store

Build a production ready mini store that uses routing, state, and data fetching.

## Features

- Product list and product details
- Cart with persistence
- Checkout flow with form validation
- Optimistic UI for cart updates
- Basic analytics events

## Suggested Structure

```
src/
├── app/
│   ├── App.tsx
│   ├── router.tsx
│   └── providers.tsx
├── features/
│   ├── cart/
│   ├── products/
│   └── checkout/
├── shared/
│   ├── ui/
│   ├── api/
│   └── utils/
└── main.tsx
```

## Cart Store (Zustand)

```tsx
import { create } from 'zustand';

type Item = { id: string; title: string; price: number; qty: number };

type CartState = {
  items: Item[];
  add: (item: Item) => void;
  updateQty: (id: string, qty: number) => void;
};

export const useCart = create<CartState>()((set, get) => ({
  items: [],
  add: (item) => set({ items: [...get().items, item] }),
  updateQty: (id, qty) =>
    set({ items: get().items.map((i) => (i.id == id ? { ...i, qty } : i)) })
}));
```

## Products Route (React Router Loader)

```tsx
import { useLoaderData } from 'react-router';

type Product = { id: string; title: string; price: number };

export async function loader() {
  const res = await fetch('/api/products');
  if (!res.ok) throw new Response('Failed', { status: 500 });
  return res.json();
}

export function ProductsRoute() {
  const products = useLoaderData() as Product[];
  return (
    <ul>
      {products.map((p) => (
        <li key={p.id}>{p.title} - ${p.price}</li>
      ))}
    </ul>
  );
}
```

## Checklist

- Add tests for cart and checkout
- Add 404 and error boundaries
- Add loading skeletons with Suspense
- Run Lighthouse and fix slow pages

---

[Previous: SSR & SSG with Next.js/Remix](./31-ssr-ssg.md) | [Back to Index](./README.md)
