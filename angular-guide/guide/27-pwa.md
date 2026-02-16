# PWA & Service Workers

Angular supports Progressive Web Apps (PWA) via the Angular service worker. This enables offline caching, background sync patterns, and installable web apps.

## Add PWA Support

```bash
ng add @angular/pwa
```

This adds:

- `manifest.webmanifest`
- `ngsw-config.json`
- Service worker registration

## Register Service Worker (Standalone)

```typescript
import { ApplicationConfig, isDevMode } from '@angular/core';
import { provideServiceWorker } from '@angular/service-worker';

export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerWhenStable:30000'
    })
  ]
};
```

## Caching Strategies

- **Asset Groups**: Cache app shell and static assets
- **Data Groups**: Cache API calls with freshness policies

Example `ngsw-config.json`:

```json
{
  "assetGroups": [
    {
      "name": "app",
      "installMode": "prefetch",
      "resources": {
        "files": ["/index.html", "/*.css", "/*.js"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "api",
      "urls": ["/api/**"],
      "cacheConfig": {
        "strategy": "freshness",
        "maxSize": 100,
        "maxAge": "1h"
      }
    }
  ]
}
```

## Best Practices

- Make offline UI states explicit
- Keep cache sizes reasonable
- Version your API and handle updates gracefully

## Next Steps

- [Architecture](./28-architecture.md) - System design patterns
- [Deployment](./29-deployment.md) - Production rollout

---

[Previous: Security Best Practices](./26-security.md) | [Back to Index](./README.md) | [Next: Architecture & Patterns](./28-architecture.md)
