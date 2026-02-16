# 01 - Introduction & Setup

Welcome to the React Frontend Guide. This chapter introduces React, highlights what is new in React 19, and sets up a modern project with Vite.

## What is React?

React is a JavaScript library for building user interfaces. It encourages a component-based architecture where UI is composed from small, reusable parts.

### Key Features

- Component-based UI with reusable building blocks
- Declarative rendering based on state
- Hooks for state, side effects, and performance
- Rich ecosystem for routing, data, styling, and testing

### What's New in React 19

- Actions for async mutations and form submissions
- useActionState, useFormStatus, useOptimistic
- use() to read resources with Suspense
- New JSX transform required for React 19 improvements
- UMD builds removed (use ESM or CDN builds)

## Recommended Stack (2026)

- Use Vite for modern SPA tooling
- Use React Router data routers for routing and mutations
- Use a framework like Next.js or Remix for full stack apps
- Create React App is deprecated for new projects

## Prerequisites

- Modern JavaScript (ES6+)
- Node.js 20.19+ or 22.12+ for Vite 7
- A code editor (VS Code recommended)

## Install Node.js (Recommended)

```bash
node --version
npm --version
```

Use nvm (Linux and macOS) if needed:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
nvm install --lts
nvm use --lts
```

## Create Your First React App (Vite)

```bash
npm create vite@latest my-react-app -- --template react-ts
cd my-react-app
npm install
npm run dev
```

Open http://localhost:5173.

## Project Structure

```
my-react-app/
├── public/
├── src/
│   ├── assets/
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── index.html
├── package.json
└── vite.config.ts
```

## Your First Component

```tsx
export function Hello() {
  return (
    <section>
      <h1>Hello React 19</h1>
      <p>Welcome to your first component.</p>
    </section>
  );
}
```

## Next Steps

- [JSX & Components](./02-jsx-components.md)
- [Props & State](./03-props-state.md)

---

[Back to Index](./README.md) | [Next: JSX & Components](./02-jsx-components.md)
