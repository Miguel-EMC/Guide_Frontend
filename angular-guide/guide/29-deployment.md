# Deployment

This guide covers building and deploying Angular applications to various platforms.

## Building for Production

### Basic Build

```bash
ng build
```

Output is in `dist/<project-name>/` folder.

### Build Configuration

**angular.json:**

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                }
              ],
              "outputHashing": "all",
              "optimization": true,
              "sourceMap": false
            },
            "staging": {
              "optimization": true,
              "sourceMap": true,
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.staging.ts"
                }
              ]
            }
          }
        }
      }
    }
  }
}
```

### Build Commands

```bash
# Production build
ng build --configuration=production

# Staging build
ng build --configuration=staging

# With base href
ng build --base-href=/my-app/

# Analyze bundle
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

## Environment Configuration

**src/environments/environment.ts:**

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
  features: {
    analytics: false,
    newDashboard: true
  }
};
```

**src/environments/environment.prod.ts:**

```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.myapp.com',
  features: {
    analytics: true,
    newDashboard: true
  }
};
```

**Using in code:**

```typescript
import { environment } from '../environments/environment';

@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = environment.apiUrl;
}
```

## Docker Deployment

### Dockerfile

```dockerfile
# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build -- --configuration=production

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist/my-app/browser /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### nginx.conf

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    server {
        listen 80;
        server_name localhost;
        root /usr/share/nginx/html;
        index index.html;

        # Gzip compression
        gzip on;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Handle Angular routing
        location / {
            try_files $uri $uri/ /index.html;
        }

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
    }
}
```

### Docker Compose

```yaml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    restart: unless-stopped
```

### Build and Run

```bash
# Build image
docker build -t my-angular-app .

# Run container
docker run -p 80:80 my-angular-app

# With docker-compose
docker-compose up -d
```

## Vercel Deployment

### vercel.json

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist/my-app/browser"
      }
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/index.html"
    }
  ]
}
```

### Deploy

```bash
# Install Vercel CLI
npm i -g vercel

# Deploy
vercel

# Production deploy
vercel --prod
```

## Netlify Deployment

### netlify.toml

```toml
[build]
  command = "npm run build"
  publish = "dist/my-app/browser"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"

[[headers]]
  for = "/*.js"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

### Deploy

```bash
# Install Netlify CLI
npm i -g netlify-cli

# Deploy preview
netlify deploy

# Production deploy
netlify deploy --prod
```

## Firebase Hosting

### Setup

```bash
# Install Firebase CLI
npm i -g firebase-tools

# Login
firebase login

# Initialize project
firebase init hosting
```

### firebase.json

```json
{
  "hosting": {
    "public": "dist/my-app/browser",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "**/*.@(js|css)",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "public, max-age=31536000, immutable"
          }
        ]
      }
    ]
  }
}
```

### Deploy

```bash
# Build
ng build --configuration=production

# Deploy
firebase deploy
```

## AWS S3 + CloudFront

### S3 Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-angular-app/*"
    }
  ]
}
```

### Deploy Script

```bash
#!/bin/bash

BUCKET_NAME="my-angular-app"
DISTRIBUTION_ID="EXXXXXXXXXX"

# Build
ng build --configuration=production

# Sync to S3
aws s3 sync dist/my-app/browser s3://$BUCKET_NAME --delete

# Invalidate CloudFront
aws cloudfront create-invalidation \
  --distribution-id $DISTRIBUTION_ID \
  --paths "/*"
```

## CI/CD with GitHub Actions

### .github/workflows/deploy.yml

```yaml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm run test -- --watch=false --browsers=ChromeHeadless

      - name: Build
        run: npm run build -- --configuration=production

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Deploy to Firebase
        uses: FirebaseExtended/action-hosting-deploy@v0
        with:
          repoToken: '${{ secrets.GITHUB_TOKEN }}'
          firebaseServiceAccount: '${{ secrets.FIREBASE_SERVICE_ACCOUNT }}'
          channelId: live
          projectId: my-project
```

## SSR Deployment

### Node.js Server

```typescript
// server.ts
import 'zone.js/node';
import { APP_BASE_HREF } from '@angular/common';
import { CommonEngine } from '@angular/ssr';
import express from 'express';
import { fileURLToPath } from 'node:url';
import { dirname, join, resolve } from 'node:path';
import bootstrap from './src/main.server';

export function app(): express.Express {
  const server = express();
  const serverDistFolder = dirname(fileURLToPath(import.meta.url));
  const browserDistFolder = resolve(serverDistFolder, '../browser');
  const indexHtml = join(serverDistFolder, 'index.server.html');

  const commonEngine = new CommonEngine();

  server.set('view engine', 'html');
  server.set('views', browserDistFolder);

  // Serve static files
  server.get('*.*', express.static(browserDistFolder, {
    maxAge: '1y'
  }));

  // All regular routes use the Angular engine
  server.get('*', (req, res, next) => {
    const { protocol, originalUrl, baseUrl, headers } = req;

    commonEngine
      .render({
        bootstrap,
        documentFilePath: indexHtml,
        url: `${protocol}://${headers.host}${originalUrl}`,
        publicPath: browserDistFolder,
        providers: [{ provide: APP_BASE_HREF, useValue: baseUrl }],
      })
      .then((html) => res.send(html))
      .catch((err) => next(err));
  });

  return server;
}

function run(): void {
  const port = process.env['PORT'] || 4000;
  const server = app();
  server.listen(port, () => {
    console.log(`Node Express server listening on http://localhost:${port}`);
  });
}

run();
```

### Dockerfile for SSR

```dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build:ssr

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 4000
CMD ["node", "dist/my-app/server/server.mjs"]
```

## Performance Optimization

### Bundle Analysis

```bash
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

### Lazy Loading

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  }
];
```

### Preloading Strategy

```typescript
provideRouter(routes, withPreloading(PreloadAllModules))
```

## Health Checks

### Health Endpoint

```typescript
// health.component.ts
@Component({
  selector: 'app-health',
  template: `<pre>{{ health | json }}</pre>`
})
export class HealthComponent {
  health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: environment.version
  };
}
```

## Summary

| Platform | Key Files |
|----------|-----------|
| Docker | `Dockerfile`, `nginx.conf`, `docker-compose.yml` |
| Vercel | `vercel.json` |
| Netlify | `netlify.toml` |
| Firebase | `firebase.json` |
| AWS | S3 policy, CloudFront config |
| CI/CD | `.github/workflows/*.yml` |

| Task | Command |
|------|---------|
| Build | `ng build --configuration=production` |
| Analyze | `ng build --stats-json` |
| Test | `ng test --watch=false` |
| Deploy | Platform-specific CLI |

## Next Steps

- [Project: Task Manager App](./30-project-task-manager.md) - Complete project example

---

[Previous: Architecture & Patterns](./28-architecture.md) | [Back to Index](./README.md) | [Next: Project: Task Manager App](./30-project-task-manager.md)
