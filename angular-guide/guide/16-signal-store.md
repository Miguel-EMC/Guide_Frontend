# Signal Store (NgRx Signals)

Signal Store is a signal-first approach to state management. It is designed for Angular apps that want store-level structure without the overhead of action reducers. Signal Store is part of the NgRx Signals ecosystem.

## When to Use Signal Store

- You want a store abstraction without full Redux ceremony
- You prefer signals over observable-based APIs
- You need shared feature state with a small team

## Installation

```bash
npm install @ngrx/signals
```

## Example Store

```typescript
import { computed } from '@angular/core';
import { signalStore, withState, withComputed, withMethods } from '@ngrx/signals';

interface CartItem {
  id: string;
  title: string;
  price: number;
  qty: number;
}

interface CartState {
  items: CartItem[];
}

export const CartStore = signalStore(
  { providedIn: 'root' },
  withState<CartState>({ items: [] }),
  withComputed(({ items }) => ({
    count: computed(() => items().length),
    total: computed(() =>
      items().reduce((sum, item) => sum + item.price * item.qty, 0)
    )
  })),
  withMethods((store) => ({
    add(item: CartItem) {
      store.items.update(items => [...items, item]);
    },
    remove(id: string) {
      store.items.update(items => items.filter(i => i.id != id));
    },
    setQty(id: string, qty: number) {
      store.items.update(items =>
        items.map(i => i.id === id ? { ...i, qty } : i)
      );
    }
  }))
);
```

## Using the Store

```typescript
import { Component, inject } from '@angular/core';
import { CartStore } from './cart.store';

@Component({
  standalone: true,
  template: `
    <h3>Items: {{ cart.count() }}</h3>
    <h3>Total: {{ cart.total() | currency }}</h3>
    <button (click)="addSample()">Add</button>
  `
})
export class CartSummaryComponent {
  cart = inject(CartStore);

  addSample() {
    this.cart.add({ id: '1', title: 'Keyboard', price: 99, qty: 1 });
  }
}
```

## RxJS Interop

When you need to consume observables, wrap them in signals using `toSignal()` or create methods that subscribe and update store signals.

## Best Practices

- Keep store state minimal and normalized
- Use computed selectors for derived values
- Encapsulate writes inside store methods
- Avoid multi-store write cycles

## Next Steps

- [Angular Signals](./12-signals.md) - Signal fundamentals
- [NgRx Store](./15-ngrx.md) - Redux-style store

---

[Previous: NgRx Store](./15-ngrx.md) | [Back to Index](./README.md) | [Next: Angular Material](./17-angular-material.md)
