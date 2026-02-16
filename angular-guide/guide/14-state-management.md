# State Management

Managing state is a crucial aspect of building complex Angular applications. As your app grows, you'll need a clear strategy for local UI state, shared feature state, and server state.

## State Layers

- **Local UI State**: Component-only data (form inputs, tabs, toggles). Use signals or component properties.
- **Feature State**: Shared within a feature area (cart, profile, filters). Use services with signals.
- **Global App State**: Cross-cutting state (auth, permissions, settings). Use a formal store.
- **Server State**: Data from APIs that needs caching, refetching, and normalization.

## Signals in Services

Signals are ideal for local or feature state in Angular v21.

```typescript
import { Injectable, computed, signal } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class CartStore {
  private items = signal<CartItem[]>([]);

  readonly itemsCount = computed(() => this.items().length);
  readonly total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.qty, 0)
  );

  add(item: CartItem) {
    this.items.update(items => [...items, item]);
  }

  remove(id: string) {
    this.items.update(items => items.filter(i => i.id !== id));
  }
}
```

## When to Use a Store Library

Use a formal store when you need:

- Time-travel debugging and devtools
- Highly predictable global state transitions
- Complex side effects and data orchestration
- A standard architecture across large teams

## Popular Options

- **NgRx Store**: Redux-style state with actions, reducers, selectors, and effects
- **Signal Store**: Signal-first store patterns (NgRx Signals)

## Best Practices

- Keep state normalized for large collections
- Separate read models from write models (CQRS-style)
- Avoid storing derived data; compute it
- Prefer feature stores over a single giant global store

## Next Steps

- [NgRx Store](./15-ngrx.md) - Redux-style state management
- [Signal Store](./16-signal-store.md) - Signal-first store architecture

---

[Previous: RxJS & Observables](./13-rxjs.md) | [Back to Index](./README.md) | [Next: NgRx Store](./15-ngrx.md)
