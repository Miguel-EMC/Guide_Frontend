# Introduction & Setup

This guide covers Angular v21 fundamentals, installation, and creating your first application.

## What is Angular?

Angular is a platform and framework for building single-page client applications using HTML and TypeScript. Developed and maintained by Google, it's one of the most popular frameworks for enterprise-scale applications.

### Key Features

| Feature | Description |
|---------|-------------|
| **Component-Based** | Build encapsulated components that manage their own state |
| **TypeScript** | Full TypeScript support with strong typing |
| **Signals** | Fine-grained reactivity for efficient updates |
| **SSR/SSG** | Server-side rendering and static site generation built-in |
| **CLI** | Powerful command-line interface for development |
| **Dependency Injection** | Hierarchical DI system for scalable applications |

### What's New in Angular v21

| Feature | Description |
|---------|-------------|
| **Signal Forms** | New streamlined, signal-based approach to forms |
| **MCP Server Tools** | AI-powered development workflow integration |
| **Angular Aria** | Enhanced accessibility features |
| **Performance Updates** | Improved hydration and rendering |
| **Standalone by Default** | Components are standalone without NgModules |

### Angular vs Other Frameworks

| Feature | Angular | React | Vue |
|---------|---------|-------|-----|
| **Type** | Full Framework | Library | Framework |
| **Language** | TypeScript | JavaScript/TS | JavaScript/TS |
| **State** | Signals/RxJS | useState/Redux | Composition API |
| **CLI** | Official CLI | Create React App | Vue CLI |
| **Learning Curve** | Steeper | Moderate | Gentler |
| **Best For** | Enterprise | Flexibility | Simplicity |

### Why Choose Angular?

1. **Enterprise Ready**: Battle-tested in large-scale applications
2. **Complete Solution**: Routing, forms, HTTP, testing included
3. **Strong Typing**: TypeScript ensures code quality
4. **Google Support**: Long-term support and regular updates
5. **Large Ecosystem**: Rich library and tool ecosystem
6. **Opinionated**: Clear patterns and conventions

## Prerequisites

Before starting, ensure you have:

- Node.js 20+ installed (LTS recommended)
- npm 10+ or yarn/pnpm
- Basic HTML, CSS, JavaScript knowledge
- A code editor (VS Code recommended)
- Terminal/command line access

## Installation

### Step 1: Install Node.js

**Verify Node.js installation:**

```bash
node --version   # Should be 20.x or higher
npm --version    # Should be 10.x or higher
```

**Install via nvm (recommended):**

```bash
# Install nvm (Linux/macOS)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# Install Node.js LTS
nvm install --lts
nvm use --lts
```

**Windows:**

```bash
# Install nvm-windows from https://github.com/coreybutler/nvm-windows
# Or use the official Node.js installer
```

### Step 2: Install Angular CLI

```bash
npm install -g @angular/cli
```

**Verify installation:**

```bash
ng version
```

Expected output:

```
Angular CLI: 21.x.x
Node: 20.x.x
Package Manager: npm 10.x.x
```

### Step 3: Configure CLI (Optional)

```bash
# Set default package manager
ng config -g cli.packageManager npm  # or yarn, pnpm

# Disable analytics (optional)
ng analytics disable

# Enable strict mode by default
ng config -g schematics.@schematics/angular:component.standalone true
```

## Your First Application

### Step 1: Create New Project

```bash
ng new my-first-app
```

**CLI prompts:**

| Prompt | Recommended Choice |
|--------|-------------------|
| Stylesheet format | SCSS |
| SSR/SSG | Yes (for production apps) |

**Quick start (skip prompts):**

```bash
ng new my-first-app --style=scss --ssr --skip-git
```

**All creation options:**

| Flag | Description |
|------|-------------|
| `--style=scss` | Use SCSS for styles |
| `--ssr` | Enable server-side rendering |
| `--routing` | Add routing module |
| `--skip-tests` | Skip test files |
| `--skip-git` | Skip git initialization |
| `--prefix=app` | Component selector prefix |
| `--strict` | Enable strict TypeScript |

### Step 2: Navigate and Serve

```bash
cd my-first-app
ng serve
```

**Serve options:**

| Flag | Description |
|------|-------------|
| `--open` or `-o` | Open browser automatically |
| `--port 4201` | Use different port |
| `--host 0.0.0.0` | Allow external access |
| `--hmr` | Enable hot module replacement |
| `--ssl` | Use HTTPS |

Open: `http://localhost:4200/`

### Step 3: Project Structure

```
my-first-app/
├── src/
│   ├── app/
│   │   ├── app.component.ts      # Root component
│   │   ├── app.component.html    # Root template
│   │   ├── app.component.scss    # Root styles
│   │   ├── app.component.spec.ts # Root tests
│   │   ├── app.config.ts         # App configuration
│   │   └── app.routes.ts         # Route definitions
│   ├── index.html                # Main HTML page
│   ├── main.ts                   # Application entry point
│   └── styles.scss               # Global styles
├── public/                       # Static assets (images, fonts)
├── angular.json                  # Angular CLI config
├── package.json                  # Dependencies
├── tsconfig.json                 # TypeScript config
├── tsconfig.app.json             # App TypeScript config
└── README.md
```

**Key Files Explained:**

| File | Purpose |
|------|---------|
| `main.ts` | Bootstraps the application |
| `app.config.ts` | Application-wide providers and configuration |
| `app.routes.ts` | Route configuration |
| `angular.json` | Build, serve, and project settings |
| `tsconfig.json` | TypeScript compiler options |

### Step 4: Understanding main.ts

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { appConfig } from './app/app.config';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, appConfig)
  .catch((err) => console.error(err));
```

### Step 5: Understanding app.config.ts

```typescript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

### Step 6: Understanding the Root Component

**app.component.ts:**

```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'my-first-app';
}
```

**Code Breakdown:**

| Element | Description |
|---------|-------------|
| `@Component` | Decorator defining component metadata |
| `selector` | CSS selector for using component (`<app-root>`) |
| `standalone: true` | Modern standalone component (no NgModule needed) |
| `imports` | Dependencies used in template |
| `templateUrl` | Path to external HTML template |
| `styleUrl` | Path to external stylesheet |

**app.component.html:**

```html
<h1>Welcome to {{ title }}!</h1>
<router-outlet />
```

## CLI Commands Reference

### General Commands

| Command | Description |
|---------|-------------|
| `ng new <name>` | Create new project |
| `ng serve` | Start dev server |
| `ng build` | Build for production |
| `ng test` | Run unit tests |
| `ng e2e` | Run end-to-end tests |
| `ng lint` | Lint code |
| `ng update` | Update Angular |

### Generate Commands

| Command | Shorthand | Creates |
|---------|-----------|---------|
| `ng generate component` | `ng g c` | Component |
| `ng generate service` | `ng g s` | Service |
| `ng generate directive` | `ng g d` | Directive |
| `ng generate pipe` | `ng g p` | Pipe |
| `ng generate guard` | `ng g g` | Route guard |
| `ng generate interceptor` | `ng g interceptor` | HTTP interceptor |
| `ng generate module` | `ng g m` | Module |
| `ng generate class` | `ng g cl` | Class |
| `ng generate interface` | `ng g i` | Interface |
| `ng generate enum` | `ng g e` | Enum |

### Add Libraries

```bash
# Add Angular Material
ng add @angular/material

# Add PWA support
ng add @angular/pwa

# Add ESLint
ng add @angular-eslint/schematics
```

## VS Code Configuration

### Recommended Extensions

| Extension | Purpose |
|-----------|---------|
| Angular Language Service | IntelliSense, errors, navigation |
| Angular Snippets | Code snippets |
| Prettier | Code formatting |
| ESLint | Code linting |
| EditorConfig | Consistent coding styles |
| Path Intellisense | Autocomplete file paths |

### settings.json

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "angular.enable-strict-mode-prompt": false
}
```

### .editorconfig

```ini
root = true

[*]
charset = utf-8
indent_style = space
indent_size = 2
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
max_line_length = off
trim_trailing_whitespace = false
```

## Common Issues

### Port Already in Use

```bash
# Use different port
ng serve --port 4201

# Or kill process on port (Linux/macOS)
lsof -ti:4200 | xargs kill -9

# Windows
netstat -ano | findstr :4200
taskkill /PID <PID> /F
```

### Module Not Found

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Or with npm cache
npm cache clean --force
npm install
```

### TypeScript Errors

```bash
# Check TypeScript version
npx tsc --version

# Should match Angular's requirements (usually 5.4+)
```

### Memory Issues During Build

```bash
# Increase Node memory limit
export NODE_OPTIONS="--max-old-space-size=8192"
ng build
```

## Summary

You learned:

- What Angular is and its key features
- What's new in Angular v21
- How to install Angular CLI
- Creating your first application
- Understanding project structure
- Essential CLI commands
- VS Code configuration

## Next Steps

- [Getting Started](./02-getting-started.md) - Create your first component
- [Components](./03-components.md) - Deep dive into components

---

[Back to Index](./README.md) | [Next: Getting Started](./02-getting-started.md)
