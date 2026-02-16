# 20 - Tailwind CSS Integration

Tailwind is a utility-first CSS framework that works well with React and Vite.

## Install (Vite)

```bash
npm install -D tailwindcss @tailwindcss/vite
```

Update vite.config.ts:

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  plugins: [react(), tailwindcss()]
});
```

Add Tailwind to your CSS:

```css
/* src/index.css */
@import "tailwindcss";
```

## Usage

```tsx
export function Hero() {
  return (
    <section className="mx-auto max-w-4xl p-8">
      <h1 className="text-3xl font-bold">React 19 + Tailwind</h1>
      <p className="mt-2 text-slate-600">Build fast UI with utility classes.</p>
      <button className="mt-4 rounded bg-black px-4 py-2 text-white">Get Started</button>
    </section>
  );
}
```

## Tips

- Use class helpers like clsx for conditional classes
- Centralize variants with tools like class-variance-authority
- Keep custom CSS small and use Tailwind for layout and spacing

## Next Steps

- [UI Component Libraries](./21-ui-libraries.md)
- [React Router](./22-react-router.md)

---

[Previous: CSS Modules & Styled Components](./19-styling.md) | [Back to Index](./README.md) | [Next: UI Component Libraries](./21-ui-libraries.md)
