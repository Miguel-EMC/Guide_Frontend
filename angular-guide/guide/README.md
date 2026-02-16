# Angular v21 Frontend Guide - Table of Contents

Welcome to the comprehensive guide to building modern, scalable web applications with Angular v21.

## What's New in v21 🚀

- **Zoneless by Default** - Better performance, smaller bundles
- **Vitest Default Test Runner** - Faster, modern testing experience  
- **Angular Aria** - 13 accessible, unstyled components (Developer Preview)
- **Signal Forms (Experimental)** - New streamlined form handling
- **MCP Server Tools** - AI-powered development workflow (Stable)
- **Enhanced Template Syntax** - Multiple switch cases, regex support

## Chapters

### Fundamentals

1.  [Introduction & Setup](./01-introduction.md) ✨ Updated for v21
2.  [Getting Started](./02-getting-started.md)
3.  [Components](./03-components.md)
4.  [Templates & Data Binding](./04-templates-data-binding.md)
5.  [Directives](./05-directives.md)
6.  [Pipes](./06-pipes.md)

### Core Features

7.  [Services & Dependency Injection](./07-services-and-dependency-injection.md)
8.  [Routing & Navigation](./08-routing.md)
9.  [Reactive Forms](./09-reactive-forms.md)
10. [Signal Forms (Experimental)](./10-signal-forms.md) ✨ Updated for v21
11. [HTTP Client](./11-http-client.md)
12. [Angular Signals](./12-signals.md)
13. [RxJS & Observables](./13-rxjs.md)

### State Management

14. [State Management](./14-state-management.md)
15. [NgRx Store](./15-ngrx.md) (Chapter not yet available)
16. [Signal Store](./16-signal-store.md) (Chapter not yet available)

### UI & Libraries

17. [Angular Material](./17-angular-material.md)
18. [Libraries and Integrations](./18-libraries-and-integrations.md)
19. [PrimeNG](./19-primeng.md) (Chapter not yet available)
20. [Tailwind CSS Integration](./20-tailwind.md) (Chapter not yet available)
21. [Animations](./21-animations.md) (Chapter not yet available)
22. **[Angular Aria (Developer Preview)](./31-angular-aria.md)** 🆕 NEW!

### Advanced Topics

23. [Testing](./22-testing.md) ✨ Updated with Vitest
24. [SSR & Hydration](./23-ssr-hydration.md)
25. [Internationalization (i18n)](./24-i18n.md) (Chapter not yet available)
26. [Performance Optimization](./25-performance.md) (Chapter not yet available)
27. [Security Best Practices](./26-security.md) (Chapter not yet available)
28. [PWA & Service Workers](./27-pwa.md) (Chapter not yet available)
29. **[Zoneless Change Detection](./32-zoneless-change-detection.md)** 🆕 NEW!

### Production

30. [Architecture & Patterns](./28-architecture.md)
31. [Deployment](./29-deployment.md)
32. [Project: Task Manager App](./30-project-task-manager.md)

---

## v21 Migration Guide

### Upgrading Existing Applications

```bash
# 1. Update to Angular 21
ng update @angular/core @angular/cli

# 2. Run migration tools
ng g @schematics/angular:onpush-zoneless-migration
ng g @schematics/angular:refactor-jasmine-vitest

# 3. Add new features
ng add @angular/aria
npm install @angular/forms  # Updated for Signal Forms
```

### Key Changes Summary

- **Zoneless**: New apps don't include Zone.js by default
- **Vitest**: Replaces Karma/Jasmine as default test runner  
- **Signal Forms**: Experimental form handling
- **Angular Aria**: 13 new accessible components
- **Dependencies**: TypeScript 5.9+, Node.js 20.19+

---

_Guide created by Miguel Muzo ([@migueldev11](https://github.com/migueldev11))_

**Last Updated**: February 2026 - Angular v21 Features
