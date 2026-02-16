# Tailwind CSS Integration

Tailwind CSS is a utility-first CSS framework that pairs well with Angular for rapid UI development.

## Installation (Tailwind v4)

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

## Configure PostCSS

Create `.postcssrc.json`:

```json
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

## Import Tailwind

Add to `src/styles.css` or `src/styles.scss`:

```css
@import "tailwindcss";
```

## Example

```html
<div class="p-6 max-w-md mx-auto bg-white rounded-xl shadow">
  <h2 class="text-xl font-semibold">Angular + Tailwind</h2>
  <p class="text-gray-600 mt-2">Build fast and consistent UIs.</p>
  <button class="mt-4 px-4 py-2 bg-black text-white rounded">
    Get Started
  </button>
</div>
```

## Best Practices

- Keep global styles minimal and rely on utilities
- Extract repeated patterns into components
- Use `@layer` for shared tokens when needed

## Next Steps

- [Animations](./21-animations.md) - Add motion to your UI
- [PrimeNG](./19-primeng.md) - UI components

---

[Previous: PrimeNG](./19-primeng.md) | [Back to Index](./README.md) | [Next: Animations](./21-animations.md)
