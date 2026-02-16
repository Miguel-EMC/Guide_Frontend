# Performance Optimization

Performance in Angular is about rendering less, doing less work per update, and shipping fewer bytes.

## High-Impact Techniques

- Use standalone components and lazy-loaded routes
- Prefer signals and `OnPush` change detection
- Use `@defer` to delay non-critical UI
- Use `NgOptimizedImage` for images
- Avoid heavy synchronous work on the main thread

## Track By in Lists

```html
@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
}
```

## Defer Non-Critical UI

```html
@defer {
  <app-heavy-widget />
} @placeholder {
  <p>Loading widget...</p>
}
```

## Lazy Routes

```typescript
import { Routes } from '@angular/router';
const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.page').then(m => m.AdminPage)
  }
];
```

## Best Practices

- Measure with Lighthouse and Web Vitals
- Avoid large JSON payloads and over-fetching
- Cache server state when possible

## Next Steps

- [Security](./26-security.md) - Secure your app
- [PWA](./27-pwa.md) - Offline-first performance

---

[Previous: Internationalization](./24-i18n.md) | [Back to Index](./README.md) | [Next: Security Best Practices](./26-security.md)
