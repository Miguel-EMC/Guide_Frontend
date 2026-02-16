# Angular v21 Frontend Guide - Table of Contents

Welcome to the comprehensive guide to building modern, scalable web applications with Angular v21.

## What's New in v21

- Zoneless by default for new applications (smaller bundles, faster runtime)
- Vitest as the default unit test runner
- Signal Forms (experimental)
- Angular Aria (developer preview)
- MCP server tools for AI-assisted workflows (stable)
- Template control flow improvements (multi-case and regex matching in @switch)
- Signals formatter in DevTools

## Version Requirements (v21)

- Node.js: ^20.19.0 || ^22.12.0 || ^24.0.0
- TypeScript: >=5.9.0 <6.0.0
- RxJS: ^6.5.3 || ^7.4.0

## Chapters

### Fundamentals

1. [Introduction & Setup](./01-introduction.md)
2. [Getting Started](./02-getting-started.md)
3. [Components](./03-components.md)
4. [Templates & Data Binding](./04-templates-data-binding.md)
5. [Directives](./05-directives.md)
6. [Pipes](./06-pipes.md)

### Core Features

7. [Services & Dependency Injection](./07-services-and-dependency-injection.md)
8. [Routing & Navigation](./08-routing.md)
9. [Reactive Forms](./09-reactive-forms.md)
10. [Signal Forms (Experimental)](./10-signal-forms.md)
11. [HTTP Client](./11-http-client.md)
12. [Angular Signals](./12-signals.md)
13. [RxJS & Observables](./13-rxjs.md)

### State Management

14. [State Management](./14-state-management.md)
15. [NgRx Store](./15-ngrx.md)
16. [Signal Store](./16-signal-store.md)

### UI & Libraries

17. [Angular Material](./17-angular-material.md)
18. [Libraries and Integrations](./18-libraries-and-integrations.md)
19. [PrimeNG](./19-primeng.md)
20. [Tailwind CSS Integration](./20-tailwind.md)
21. [Animations](./21-animations.md)
22. [Angular Aria (Developer Preview)](./31-angular-aria.md)

### Advanced Topics

23. [Testing](./22-testing.md)
24. [SSR & Hydration](./23-ssr-hydration.md)
25. [Internationalization (i18n)](./24-i18n.md)
26. [Performance Optimization](./25-performance.md)
27. [Security Best Practices](./26-security.md)
28. [PWA & Service Workers](./27-pwa.md)
29. [Zoneless Change Detection](./32-zoneless-change-detection.md)

### Production

30. [Architecture & Patterns](./28-architecture.md)
31. [Deployment](./29-deployment.md)
32. [Project: Task Manager App](./30-project-task-manager.md)
33. [Enterprise Patterns](./33-enterprise-patterns.md)

---

## v21 Migration Guide

```bash
# Update core and CLI
ng update @angular/core @angular/cli

# Optional: update Angular Material
ng update @angular/material

# Optional: add Angular Aria
npm install @angular/aria
```

### Migration Checklist

- Verify Node.js and TypeScript versions meet v21 requirements
- Review breaking changes and migrations in `ng update`
- If you rely on Zone.js, keep it explicitly (see the zoneless chapter)
- Adopt Vitest (default) and update any Jasmine/Karma tooling
- Evaluate Signal Forms and Angular Aria before production use

---

_Guide created by Miguel Muzo ([@migueldev11](https://github.com/migueldev11))_

**Last Updated**: February 2026 - Angular v21
