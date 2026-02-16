# Zoneless Change Detection (v21)

Angular v21 makes zoneless change detection the default for new applications. This eliminates Zone.js overhead and makes change detection more explicit and predictable.

## Why Zoneless?

- Smaller bundles (no Zone.js)
- Faster runtime
- Fewer "magic" updates
- Cleaner stack traces

## How It Works

Zoneless apps rely on explicit reactivity:

- Signals update the view automatically
- Template events run change detection for their component
- Async work can notify Angular via signals or manual triggers

## Opting In Explicitly

You can explicitly enable zoneless change detection with `provideZonelessChangeDetection()`:

```typescript
import { ApplicationConfig, provideZonelessChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZonelessChangeDetection(),
    provideRouter([])
  ]
};
```

## Keeping Zone.js (Legacy Apps)

If you need Zone.js for legacy libraries or older patterns, keep it explicitly:

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true })
  ]
};
```

## Migration Tips

1. Prefer signals over `setTimeout`/`Promise` side effects
2. Use `computed` for derived state
3. Replace manual `ChangeDetectorRef.detectChanges()` with signals
4. Audit third-party libraries for Zone.js reliance

## Testing with Zoneless Apps

Vitest and TestBed work the same. If you rely on fakeAsync/tick, make sure the tested code does not depend on Zone.js behavior.

## Best Practices

- Use signals for component and feature state
- Prefer `async` pipe or `toSignal()` for observable interop
- Avoid global mutable state

## Next Steps

- [Signals](./12-signals.md) - Core reactivity primitives
- [Testing](./22-testing.md) - Vitest and testing patterns

---

[Previous: Angular Aria](./31-angular-aria.md) | [Back to Index](./README.md) | [Next: Enterprise Patterns](./33-enterprise-patterns.md)
