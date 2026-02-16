# Animations

Angular provides a powerful animation system based on the Web Animations API. Animations can be used for components, routes, and lists.

## Setup

Enable animations in `app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideAnimations } from '@angular/platform-browser/animations';

export const appConfig: ApplicationConfig = {
  providers: [provideAnimations()]
};
```

## Basic Fade Animation

```typescript
import { Component } from '@angular/core';
import { trigger, transition, style, animate } from '@angular/animations';

@Component({
  standalone: true,
  template: `
    <div @fade class="card">Animated card</div>
  `,
  animations: [
    trigger('fade', [
      transition(':enter', [
        style({ opacity: 0 }),
        animate('200ms ease-out', style({ opacity: 1 }))
      ]),
      transition(':leave', [
        animate('150ms ease-in', style({ opacity: 0 }))
      ])
    ])
  ]
})
export class FadeComponent {}
```

## List Stagger

```typescript
import { trigger, transition, style, animate, query, stagger } from '@angular/animations';

export const listAnimation = trigger('list', [
  transition(':enter', [
    query('li', [
      style({ opacity: 0, transform: 'translateY(8px)' }),
      stagger(40, [animate('160ms ease-out', style({ opacity: 1, transform: 'none' }))])
    ])
  ])
]);
```

## Best Practices

- Keep animations subtle and consistent
- Avoid animating layout-heavy properties
- Use `prefers-reduced-motion` for accessibility

## Next Steps

- [Testing](./22-testing.md) - Testing animated components
- [Performance Optimization](./25-performance.md) - Animation performance

---

[Previous: Tailwind CSS Integration](./20-tailwind.md) | [Back to Index](./README.md) | [Next: Testing](./22-testing.md)
