# 29 - Build Tools (Vite, Webpack)

Build tools handle bundling, dev server, and production builds.

## Vite (Recommended)

```bash
npm run dev
npm run build
npm run preview
```

Vite 7 requires Node 20.19+ or 22.12+.

## Configure Aliases

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { fileURLToPath, URL } from 'node:url';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
});
```

## Webpack and Other Bundlers

- Use Webpack or Rspack for advanced custom setups
- Ensure JSX transform is set to react-jsx
- Keep dependencies updated to match React 19

## Next Steps

- [Deployment Strategies](./30-deployment.md)
- [SSR & SSG with Next.js/Remix](./31-ssr-ssg.md)

---

[Previous: End-to-End Testing](./28-e2e-testing.md) | [Back to Index](./README.md) | [Next: Deployment Strategies](./30-deployment.md)
