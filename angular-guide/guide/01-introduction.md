# Introduction & Setup

This guide covers Angular v21 fundamentals, installation, and creating your first application.

## What is Angular?

Angular is a platform and framework for building single-page applications using HTML and TypeScript. It is maintained by Google and designed for large-scale, long-lived web applications.

### Key Features

| Feature | Description |
|---------|-------------|
| Component Model | Encapsulated UI with clear inputs/outputs |
| TypeScript First | Strong typing and tooling support |
| Signals | Fine-grained reactivity for efficient updates |
| SSR/SSG | Built-in server rendering and prerendering |
| CLI | Powerful code generation and tooling |
| DI | Hierarchical dependency injection |

### What's New in Angular v21

| Feature | Description |
|---------|-------------|
| Zoneless by Default | New apps run without Zone.js by default |
| Vitest Default Runner | Faster unit testing out of the box |
| Signal Forms (Experimental) | Signal-based forms API |
| Angular Aria (Dev Preview) | Headless, accessible UI primitives |
| MCP Server Tools (Stable) | AI-assisted workflow integration |
| Template Syntax Enhancements | Multi-case and regex support in @switch |
| Signals Formatter | Improved DevTools signal display |

## Why Choose Angular?

1. Enterprise ready and long-term supported
2. Full-stack framework with batteries included
3. Strong conventions and tooling
4. Excellent performance with signals and zoneless defaults
5. A mature ecosystem and community

## Prerequisites (v21)

- Node.js ^20.19.0 or ^22.12.0 or ^24.0.0
- TypeScript >=5.9.0 <6.0.0 (installed by Angular)
- RxJS ^6.5.3 or ^7.4.0
- A code editor (VS Code recommended)
- Terminal access

## Install Angular CLI

```bash
npm install -g @angular/cli@latest
ng version
```

## Create Your First App

```bash
ng new my-first-app --style=scss --ssr --strict
cd my-first-app
ng serve
```

Open `http://localhost:4200/`.

### Modern Project Structure

```
my-first-app/
├── src/
│   ├── app/
│   │   ├── app.component.ts
│   │   ├── app.component.html
│   │   ├── app.component.scss
│   │   ├── app.config.ts
│   │   └── app.routes.ts
│   ├── main.ts
│   └── index.html
├── angular.json
└── package.json
```

### Modern Defaults to Know

- Standalone components and functional providers
- Built-in control flow (`@if`, `@for`, `@switch`)
- Signals-first reactivity
- Zoneless by default in new projects

## Next Steps

- [Getting Started](./02-getting-started.md) - Build your first components
- [Components](./03-components.md) - Learn component fundamentals

---

[Back to Index](./README.md) | [Next: Getting Started](./02-getting-started.md)
