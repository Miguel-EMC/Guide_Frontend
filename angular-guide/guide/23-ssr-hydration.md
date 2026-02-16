# SSR & Hydration

Server-Side Rendering (SSR) improves performance and SEO by rendering Angular applications on the server.

## Overview

| Feature | Description |
|---------|-------------|
| **SSR** | Render on server, send HTML to client |
| **SSG** | Pre-render at build time |
| **Hydration** | Attach JS to server-rendered HTML |
| **Partial Hydration** | Hydrate only interactive parts |

## Setup

### New Project with SSR

```bash
ng new my-app --ssr
```

### Add SSR to Existing Project

```bash
ng add @angular/ssr
```

This adds:
- `server.ts` - Express server
- `src/main.server.ts` - Server bootstrap
- `src/app/app.config.server.ts` - Server config

## Project Structure

```
src/
├── app/
│   ├── app.config.ts           # Client config
│   ├── app.config.server.ts    # Server config
│   └── app.routes.ts
├── main.ts                     # Client entry
├── main.server.ts              # Server entry
└── index.html
server.ts                       # Express server
```

## Configuration

### app.config.ts (Client)

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideClientHydration } from '@angular/platform-browser';
import { provideHttpClient, withFetch } from '@angular/common/http';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideClientHydration(),
    provideHttpClient(withFetch())
  ]
};
```

### app.config.server.ts

```typescript
import { mergeApplicationConfig, ApplicationConfig } from '@angular/core';
import { provideServerRendering } from '@angular/platform-server';
import { appConfig } from './app.config';

const serverConfig: ApplicationConfig = {
  providers: [
    provideServerRendering()
  ]
};

export const config = mergeApplicationConfig(appConfig, serverConfig);
```

### main.server.ts

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { config } from './app/app.config.server';

const bootstrap = () => bootstrapApplication(AppComponent, config);
export default bootstrap;
```

## Hydration

### Enable Hydration

```typescript
import { provideClientHydration } from '@angular/platform-browser';
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration()
  ]
};
```

### Hydration Options

```typescript
import { ApplicationConfig } from '@angular/core';
import {
  provideClientHydration,
  withEventReplay,
  withNoHttpTransferCache,
  withHttpTransferCacheOptions
} from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(
      withEventReplay(),  // Replay events before hydration
      withHttpTransferCacheOptions({
        includeHeaders: ['X-Custom-Header'],
        includePostRequests: true
      })
    )
  ]
};
```

### Skip Hydration

For components that shouldn't hydrate:

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-ad-banner',
  host: { 'ngSkipHydration': 'true' },  // Skip hydration
  template: `<div class="ad">Ad content</div>`
})
export class AdBannerComponent {}
```

Or in template:

```html
<app-ad-banner ngSkipHydration />
```

## HTTP Transfer State

Automatically transfers HTTP responses from server to client:

```typescript
import { Component, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
// This request is made on server
// Response is transferred to client, no duplicate request
@Component({
  template: `
    @for (user of users(); track user.id) {
      <div>{{ user.name }}</div>
    }
  `
})
export class UsersComponent {
  private http = inject(HttpClient);
  users = signal<User[]>([]);

  constructor() {
    this.http.get<User[]>('/api/users').subscribe(users => {
      this.users.set(users);
    });
  }
}
```

## Platform Detection

### isPlatformBrowser / isPlatformServer

```typescript
import { Component, PLATFORM_ID, inject } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

@Component({...})
export class SafeComponent {
  private platformId = inject(PLATFORM_ID);

  ngOnInit() {
    if (isPlatformBrowser(this.platformId)) {
      // Browser-only code
      window.addEventListener('scroll', this.onScroll);
      localStorage.getItem('key');
    }

    if (isPlatformServer(this.platformId)) {
      // Server-only code
      console.log('Rendering on server');
    }
  }
}
```

### afterNextRender / afterRender

```typescript
import { Component, afterNextRender, afterRender } from '@angular/core';

@Component({...})
export class ChartComponent {
  constructor() {
    // Run once after first render (browser only)
    afterNextRender(() => {
      this.initChart();
    });

    // Run after every render (browser only)
    afterRender(() => {
      this.updateChart();
    });
  }

  initChart() {
    // Safe to use browser APIs here
    const canvas = document.getElementById('chart');
    new Chart(canvas, {...});
  }
}
```

## Static Site Generation (SSG)

### Configure Prerendering

**angular.json:**

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "options": {
            "prerender": true
          }
        }
      }
    }
  }
}
```

### Prerender Routes

```typescript
// app.routes.server.ts
import { RenderMode, ServerRoute } from '@angular/ssr';

export const serverRoutes: ServerRoute[] = [
  {
    path: '',
    renderMode: RenderMode.Prerender
  },
  {
    path: 'about',
    renderMode: RenderMode.Prerender
  },
  {
    path: 'products/:id',
    renderMode: RenderMode.Server  // Dynamic SSR
  },
  {
    path: 'admin/**',
    renderMode: RenderMode.Client  // Client-only
  }
];
```

### Prerender with Parameters

```typescript
export const serverRoutes: ServerRoute[] = [
  {
    path: 'products/:id',
    renderMode: RenderMode.Prerender,
    async getPrerenderParams() {
      const products = await fetchProducts();
      return products.map(p => ({ id: p.id.toString() }));
    }
  }
];
```

## SEO

### Meta Tags

```typescript
import { Component, inject, effect } from '@angular/core';
import { Meta, Title } from '@angular/platform-browser';

@Component({...})
export class ProductComponent {
  private meta = inject(Meta);
  private title = inject(Title);
  product = input.required<Product>();

  constructor() {
    effect(() => {
      const product = this.product();
      this.title.setTitle(`${product.name} | My Store`);
      this.meta.updateTag({ name: 'description', content: product.description });
      this.meta.updateTag({ property: 'og:title', content: product.name });
      this.meta.updateTag({ property: 'og:image', content: product.image });
    });
  }
}
```

### Resolve Title

```typescript
import { Injectable, inject } from '@angular/core';
import { Routes } from '@angular/router';
import { map } from 'rxjs/operators';
import { Observable } from 'rxjs';
export const routes: Routes = [
  {
    path: 'products/:id',
    component: ProductComponent,
    title: ProductTitleResolver
  }
];

@Injectable({ providedIn: 'root' })
export class ProductTitleResolver implements Resolve<string> {
  resolve(route: ActivatedRouteSnapshot): Observable<string> {
    const id = route.paramMap.get('id');
    return inject(ProductService).getProduct(id!).pipe(
      map(product => `${product.name} | My Store`)
    );
  }
}
```

## Caching

### Server-Side Caching

```typescript
// server.ts
import { LRUCache } from 'lru-cache';

const cache = new LRUCache<string, string>({
  max: 500,
  ttl: 1000 * 60 * 5 // 5 minutes
});

server.get('*', async (req, res, next) => {
  const cacheKey = req.originalUrl;

  // Check cache
  const cached = cache.get(cacheKey);
  if (cached) {
    return res.send(cached);
  }

  // Render
  try {
    const html = await commonEngine.render({...});
    cache.set(cacheKey, html);
    res.send(html);
  } catch (err) {
    next(err);
  }
});
```

### Cache Headers

```typescript
server.get('*', async (req, res, next) => {
  const html = await commonEngine.render({...});

  // Cache static pages
  if (isStaticPage(req.originalUrl)) {
    res.set('Cache-Control', 'public, max-age=3600'); // 1 hour
  }

  res.send(html);
});
```

## Common Issues

### Window/Document Not Defined

```typescript
import { Component } from '@angular/core';
// ❌ Bad - crashes on server
@Component({...})
export class BadComponent {
  width = window.innerWidth;  // Error on server!
}

// ✅ Good - check platform
@Component({...})
export class GoodComponent {
  width = 0;

  constructor() {
    afterNextRender(() => {
      this.width = window.innerWidth;
    });
  }
}
```

### LocalStorage

```typescript
import { Injectable, inject } from '@angular/core';
// ❌ Bad
const token = localStorage.getItem('token');

// ✅ Good
@Injectable({ providedIn: 'root' })
export class StorageService {
  private platformId = inject(PLATFORM_ID);

  getItem(key: string): string | null {
    if (isPlatformBrowser(this.platformId)) {
      return localStorage.getItem(key);
    }
    return null;
  }
}
```

### Third-Party Libraries

```typescript
import { Component, inject } from '@angular/core';
@Component({...})
export class ChartComponent {
  private platformId = inject(PLATFORM_ID);

  constructor() {
    afterNextRender(async () => {
      // Dynamically import browser-only library
      const { Chart } = await import('chart.js');
      this.initChart(Chart);
    });
  }
}
```

## Build Commands

```bash
# Development with SSR
ng serve

# Build for production
ng build

# Build SSR only
ng build --configuration=production

# Run production server
node dist/my-app/server/server.mjs
```

## Summary

| Feature | Purpose |
|---------|---------|
| `provideClientHydration()` | Enable hydration |
| `withEventReplay()` | Replay events before hydration |
| `ngSkipHydration` | Skip hydration for component |
| `afterNextRender()` | Run after first browser render |
| `isPlatformBrowser()` | Check if running in browser |
| `RenderMode.Prerender` | Pre-render at build time |
| `RenderMode.Server` | Render on each request |
| `RenderMode.Client` | Client-only rendering |

## Next Steps

- [Internationalization (i18n)](./24-i18n.md) - Multi-language support
- [Performance Optimization](./25-performance.md) - Optimize your app

---

[Previous: Testing](./22-testing.md) | [Back to Index](./README.md) | [Next: Internationalization](./24-i18n.md)
