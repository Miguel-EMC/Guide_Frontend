# 16 - Zustand

Zustand is a minimal global state library with a simple API and good performance.

## Install

```bash
npm i zustand
```

## Store Example

```tsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

type CartItem = { id: string; name: string; price: number };

type CartState = {
  items: CartItem[];
  add: (item: CartItem) => void;
  remove: (id: string) => void;
  clear: () => void;
};

export const useCart = create<CartState>()(
  devtools(
    persist(
      (set, get) => ({
        items: [],
        add: (item) => set({ items: [...get().items, item] }),
        remove: (id) => set({ items: get().items.filter((i) => i.id !== id) }),
        clear: () => set({ items: [] })
      }),
      { name: 'cart' }
    )
  )
);
```

## Selectors

```tsx
const count = useCart((s) => s.items.length);
const total = useCart((s) => s.items.reduce((sum, item) => sum + item.price, 0));
```

## Tips

- Keep actions inside the store for predictable updates
- Use selectors to avoid unnecessary renders
- Split stores by domain to keep them focused

## Next Steps

- [Jotai & Recoil](./17-jotai-recoil.md)
- [TanStack Query](./18-tanstack-query.md)

---

[Previous: State Management](./15-state-management.md) | [Back to Index](./README.md) | [Next: Jotai & Recoil](./17-jotai-recoil.md)
