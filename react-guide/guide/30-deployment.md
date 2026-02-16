# 30 - Deployment Strategies (Vercel, Netlify)

Deploying a React app depends on whether it is a static SPA or a server rendered app.

## SPA Deployment

```bash
npm run build
```

Deploy the dist folder to a static host like Netlify or Vercel.

## SPA Routing Fix (Static Hosts)

Create a rewrite so all routes serve index.html.

```
/* /index.html 200
```

## Environment Variables

- Vite exposes variables with the VITE_ prefix
- Keep secrets on the server, not in client code

## Next Steps

- [SSR & SSG with Next.js/Remix](./31-ssr-ssg.md)
- [Project: E-commerce Store](./32-e-commerce-store.md)

---

[Previous: Build Tools](./29-build-tools.md) | [Back to Index](./README.md) | [Next: SSR & SSG with Next.js/Remix](./31-ssr-ssg.md)
