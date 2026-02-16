# React v19 Frontend Guide - Table of Contents

Welcome to the comprehensive guide to building modern, scalable web applications with React v19. This guide uses Vite + TypeScript by default and stays framework-agnostic unless noted.

## What's New in React 19

- Actions for async mutations and form submissions
- useActionState, useFormStatus, useOptimistic
- use() to read resources with Suspense
- New React DOM static APIs: prerender and prerenderToNodeStream
- New JSX transform required
- UMD builds removed (use ESM or CDN builds)

## Tooling and Requirements

- React: v19.x
- Node.js: 20.19+ or 22.12+ for Vite 7
- TypeScript: recommended

## React Team Guidance

- Create React App is deprecated for new apps
- Start SPAs with Vite and a modern router
- For full stack, use a React framework like Next.js or Remix

## Chapters

### Fundamentals

1.  [Introduction & Setup](./01-introduction.md)
2.  [JSX & Components](./02-jsx-components.md)
3.  [Props & State](./03-props-state.md)
4.  [Conditional & List Rendering](./04-rendering.md)
5.  [Handling Events](./05-events.md)
6.  [Forms](./06-forms.md)

### Core Features (Hooks)

7.  [useState & useEffect](./07-usestate-useeffect.md)
8.  [useContext & useReducer](./08-usecontext-usereducer.md)
9.  [useRef & Custom Hooks](./09-useref-custom-hooks.md)
10. [useMemo, useCallback & Optimizations](./10-optimizations.md)

### Advanced Topics

11. [Context API in Depth](./11-context-api.md)
12. [Portals & Error Boundaries](./12-portals-error-boundaries.md)
13. [Concurrent React (Suspense, useTransition)](./13-concurrent-react.md)
14. [TypeScript with React](./14-typescript.md)

### State Management

15. [Introduction to State Management](./15-state-management.md)
16. [Zustand](./16-zustand.md)
17. [Jotai & Recoil](./17-jotai-recoil.md)
18. [TanStack Query](./18-tanstack-query.md)

### UI & Styling

19. [CSS Modules & Styled Components](./19-styling.md)
20. [Tailwind CSS Integration](./20-tailwind.md)
21. [UI Component Libraries](./21-ui-libraries.md)

### Routing & Navigation

22. [React Router](./22-react-router.md)

### Data Fetching

23. [Basic Data Fetching (Fetch, Axios)](./23-data-fetching.md)
24. [Advanced Data Fetching (SWR, React Query)](./24-advanced-data-fetching.md)

### Performance & Optimization

25. [Code Splitting & Lazy Loading](./25-code-splitting.md)
26. [Performance Best Practices](./26-performance.md)

### Testing

27. [Unit Testing with RTL/Vitest](./27-testing.md)
28. [End-to-End Testing](./28-e2e-testing.md)

### Deployment

29. [Build Tools (Vite, Webpack)](./29-build-tools.md)
30. [Deployment Strategies (Vercel, Netlify)](./30-deployment.md)
31. [SSR & SSG with Next.js/Remix](./31-ssr-ssg.md)

### Project

32. [Project: E-commerce Store](./32-e-commerce-store.md)

---

## React 19 Migration Checklist

- Upgrade to React 18.3 first (warnings and codemods)
- Install React 19 and React DOM 19
- Update @types/react and @types/react-dom if using TypeScript
- Ensure the new JSX transform is enabled
- Replace any UMD usage with ESM or CDN builds
- Migrate away from react-test-renderer/shallow to RTL

---

_Guide created by Miguel Muzo (@migueldev11)_

**Last Updated**: February 16, 2026 - React v19
