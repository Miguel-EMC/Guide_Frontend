# Routing & Navigation

Angular Router enables navigation between views in single-page applications without full page reloads.

## Setup

### Configure Router

**app.config.ts:**

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideRouter, withViewTransitions } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withViewTransitions())
  ]
};
```

### Define Routes

**app.routes.ts:**

```typescript
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'products', component: ProductListComponent },
  { path: 'products/:id', component: ProductDetailComponent },
  { path: '**', component: NotFoundComponent }
];
```

### Router Outlet

**app.component.html:**

```html
<nav>
  <a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">Home</a>
  <a routerLink="/about" routerLinkActive="active">About</a>
  <a routerLink="/products" routerLinkActive="active">Products</a>
</nav>

<main>
  <router-outlet />
</main>
```

## Route Configuration

### Basic Routes

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  // Static path
  { path: 'home', component: HomeComponent },

  // Empty path (default)
  { path: '', redirectTo: '/home', pathMatch: 'full' },

  // Route with data
  { path: 'about', component: AboutComponent, title: 'About Us' },

  // Wildcard (404)
  { path: '**', component: NotFoundComponent }
];
```

### Route Parameters

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  // Required parameter
  { path: 'users/:id', component: UserDetailComponent },

  // Multiple parameters
  { path: 'products/:category/:id', component: ProductComponent },

  // Optional with query params
  { path: 'search', component: SearchComponent }
];
```

### Child Routes

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  {
    path: 'admin',
    component: AdminLayoutComponent,
    children: [
      { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
      { path: 'dashboard', component: AdminDashboardComponent },
      { path: 'users', component: AdminUsersComponent },
      { path: 'settings', component: AdminSettingsComponent }
    ]
  }
];
```

**admin-layout.component.html:**

```html
<div class="admin-layout">
  <aside>
    <nav>
      <a routerLink="dashboard" routerLinkActive="active">Dashboard</a>
      <a routerLink="users" routerLinkActive="active">Users</a>
      <a routerLink="settings" routerLinkActive="active">Settings</a>
    </nav>
  </aside>
  <main>
    <router-outlet />
  </main>
</div>
```

### Named Outlets

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  { path: 'home', component: HomeComponent },
  { path: 'sidebar', component: SidebarComponent, outlet: 'sidebar' },
  { path: 'chat', component: ChatComponent, outlet: 'popup' }
];
```

```html
<router-outlet />
<router-outlet name="sidebar" />
<router-outlet name="popup" />

<!-- Navigate to named outlet -->
<a [routerLink]="[{ outlets: { popup: ['chat'] } }]">Open Chat</a>
```

## Navigation

### RouterLink Directive

```html
<!-- Basic navigation -->
<a routerLink="/home">Home</a>
<a [routerLink]="['/home']">Home</a>

<!-- With parameters -->
<a [routerLink]="['/users', userId]">User {{ userId }}</a>
<a [routerLink]="['/products', product.category, product.id]">{{ product.name }}</a>

<!-- With query parameters -->
<a [routerLink]="['/search']" [queryParams]="{ q: searchTerm, page: 1 }">Search</a>

<!-- Preserve query params -->
<a routerLink="/next" queryParamsHandling="preserve">Next</a>
<a routerLink="/next" queryParamsHandling="merge">Next</a>

<!-- Fragment -->
<a [routerLink]="['/page']" fragment="section2">Go to Section 2</a>

<!-- Relative navigation -->
<a routerLink="../sibling">Sibling Route</a>
<a routerLink="./child">Child Route</a>
```

### RouterLinkActive

```html
<!-- Add class when active -->
<a routerLink="/home" routerLinkActive="active">Home</a>

<!-- Multiple classes -->
<a routerLink="/products" routerLinkActive="active highlighted">Products</a>

<!-- Exact match only -->
<a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }">Home</a>

<!-- Template reference -->
<a routerLink="/admin" routerLinkActive #rla="routerLinkActive">
  Admin {{ rla.isActive ? '(current)' : '' }}
</a>
```

### Programmatic Navigation

```typescript
import { Component, inject } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';

@Component({...})
export class NavigationComponent {
  private router = inject(Router);
  private route = inject(ActivatedRoute);

  // Navigate to path
  goToHome() {
    this.router.navigate(['/home']);
  }

  // Navigate with parameters
  goToUser(id: number) {
    this.router.navigate(['/users', id]);
  }

  // Navigate with query params
  search(term: string) {
    this.router.navigate(['/search'], {
      queryParams: { q: term, page: 1 }
    });
  }

  // Relative navigation
  goToChild() {
    this.router.navigate(['child'], { relativeTo: this.route });
  }

  // Navigate by URL
  goByUrl() {
    this.router.navigateByUrl('/products/electronics/123?ref=home');
  }

  // Replace history
  replace() {
    this.router.navigate(['/new-page'], { replaceUrl: true });
  }

  // Skip location change
  silentNavigate() {
    this.router.navigate(['/page'], { skipLocationChange: true });
  }
}
```

## Reading Route Data

### Route Parameters

```typescript
import { Component, inject, input } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { map } from 'rxjs/operators';
import { Observable } from 'rxjs';

@Component({...})
export class UserDetailComponent {
  private route = inject(ActivatedRoute);

  // Snapshot (one-time read)
  userId = this.route.snapshot.paramMap.get('id');

  // Observable (reactive)
  userId$ = this.route.paramMap.pipe(
    map(params => params.get('id'))
  );

  // Using input() with withComponentInputBinding
  id = input<string>();  // Automatically bound from route param
}
```

### Query Parameters

```typescript
import { Component, inject, input } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { Observable } from 'rxjs';
@Component({...})
export class SearchComponent {
  private route = inject(ActivatedRoute);

  // Snapshot
  searchTerm = this.route.snapshot.queryParamMap.get('q');
  page = this.route.snapshot.queryParamMap.get('page');

  // Observable
  queryParams$ = this.route.queryParamMap;

  // Using input()
  q = input<string>();
  page = input<number>();
}
```

### Route Data

```typescript
import { Component, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
// Route configuration
{
  path: 'admin',
  component: AdminComponent,
  data: { role: 'admin', title: 'Admin Panel' }
}

// Component
@Component({...})
export class AdminComponent {
  private route = inject(ActivatedRoute);

  role = this.route.snapshot.data['role'];
  title = this.route.snapshot.data['title'];
}
```

### Component Input Binding

**app.config.ts:**

```typescript
import { provideRouter } from '@angular/router';
import { ApplicationConfig } from '@angular/core';
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withComponentInputBinding())
  ]
};
```

**Component:**

```typescript
import { Component, input } from '@angular/core';
@Component({...})
export class ProductDetailComponent {
  // Automatically bound from route param :id
  id = input.required<string>();

  // Query params
  ref = input<string>();

  // Route data
  title = input<string>();
}
```

## Route Guards

### CanActivate

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  // Redirect to login
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// Usage in routes
{
  path: 'dashboard',
  component: DashboardComponent,
  canActivate: [authGuard]
}
```

### CanActivateChild

```typescript
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';
export const adminGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  return authService.isAdmin();
};

// Protect all child routes
{
  path: 'admin',
  canActivateChild: [adminGuard],
  children: [
    { path: 'users', component: AdminUsersComponent },
    { path: 'settings', component: AdminSettingsComponent }
  ]
}
```

### CanDeactivate

```typescript
import { Component, signal } from '@angular/core';
import { Observable } from 'rxjs';
export interface CanDeactivateComponent {
  canDeactivate: () => boolean | Observable<boolean>;
}

export const unsavedChangesGuard: CanDeactivateFn<CanDeactivateComponent> = (component) => {
  if (component.canDeactivate()) {
    return true;
  }

  return confirm('You have unsaved changes. Leave anyway?');
};

// Component implementation
@Component({...})
export class EditFormComponent implements CanDeactivateComponent {
  hasUnsavedChanges = signal(false);

  canDeactivate(): boolean {
    return !this.hasUnsavedChanges();
  }
}

// Route
{
  path: 'edit/:id',
  component: EditFormComponent,
  canDeactivate: [unsavedChangesGuard]
}
```

### CanMatch

```typescript
import { inject } from '@angular/core';
export const featureFlagGuard: CanMatchFn = (route, segments) => {
  const featureService = inject(FeatureService);
  return featureService.isEnabled('new-feature');
};

{
  path: 'new-feature',
  canMatch: [featureFlagGuard],
  component: NewFeatureComponent
}
```

### Resolve

Pre-fetch data before route activation:

```typescript
import { Component, inject, input } from '@angular/core';
import { ResolveFn, ActivatedRoute } from '@angular/router';

export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;
  return userService.getUser(id);
};

// Route
{
  path: 'users/:id',
  component: UserDetailComponent,
  resolve: { user: userResolver }
}

// Component
@Component({...})
export class UserDetailComponent {
  private route = inject(ActivatedRoute);

  user = this.route.snapshot.data['user'] as User;
  // Or with input binding
  user = input<User>();
}
```

## Lazy Loading

### Lazy Load Routes

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  { path: '', component: HomeComponent },

  // Lazy load module
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES)
  },

  // Lazy load component
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component').then(m => m.DashboardComponent)
  }
];
```

**admin/admin.routes.ts:**

```typescript
import { Routes } from '@angular/router';
export const ADMIN_ROUTES: Routes = [
  { path: '', component: AdminLayoutComponent, children: [
    { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
    { path: 'dashboard', loadComponent: () => import('./dashboard.component').then(m => m.DashboardComponent) },
    { path: 'users', loadComponent: () => import('./users.component').then(m => m.UsersComponent) }
  ]}
];
```

### Preloading Strategies

```typescript
import { provideRouter, withPreloading, PreloadAllModules } from '@angular/router';
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
};
```

**Custom Preloading:**

```typescript
import { Injectable } from '@angular/core';
import { PreloadingStrategy, Route, provideRouter } from '@angular/router';
import { Observable, of } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class SelectivePreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    if (route.data?.['preload']) {
      return load();
    }
    return of(null);
  }
}

// Route configuration
{
  path: 'dashboard',
  loadComponent: () => import('./dashboard.component'),
  data: { preload: true }
}

// App config
provideRouter(routes, withPreloading(SelectivePreloadingStrategy))
```

## Router Events

```typescript
import { Component, inject } from '@angular/core';
import { Router } from '@angular/router';
@Component({...})
export class AppComponent {
  private router = inject(Router);

  constructor() {
    this.router.events.pipe(
      takeUntilDestroyed()
    ).subscribe(event => {
      if (event instanceof NavigationStart) {
        console.log('Navigation started');
      }
      if (event instanceof NavigationEnd) {
        console.log('Navigation ended', event.url);
      }
      if (event instanceof NavigationError) {
        console.error('Navigation error', event.error);
      }
      if (event instanceof NavigationCancel) {
        console.log('Navigation cancelled');
      }
    });
  }
}
```

### Loading Indicator

```typescript
import { Component, inject, signal } from '@angular/core';
import { Router } from '@angular/router';
@Component({
  template: `
    @if (isLoading()) {
      <app-loading-bar />
    }
    <router-outlet />
  `
})
export class AppComponent {
  private router = inject(Router);
  isLoading = signal(false);

  constructor() {
    this.router.events.pipe(takeUntilDestroyed()).subscribe(event => {
      if (event instanceof NavigationStart) {
        this.isLoading.set(true);
      }
      if (event instanceof NavigationEnd ||
          event instanceof NavigationError ||
          event instanceof NavigationCancel) {
        this.isLoading.set(false);
      }
    });
  }
}
```

## View Transitions

Enable smooth page transitions:

```typescript
import { provideRouter, withViewTransitions } from '@angular/router';
provideRouter(routes, withViewTransitions())
```

**Customize with CSS:**

```css
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 0.3s;
}

::view-transition-old(root) {
  animation: fade-out 0.3s ease-out;
}

::view-transition-new(root) {
  animation: fade-in 0.3s ease-in;
}

@keyframes fade-out {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}
```

## Router State

```typescript
import { Component, inject, computed } from '@angular/core';
import { Router, ActivatedRoute } from '@angular/router';
@Component({...})
export class BreadcrumbComponent {
  private router = inject(Router);
  private route = inject(ActivatedRoute);

  // Current URL
  currentUrl = this.router.url;

  // Build breadcrumbs from route tree
  breadcrumbs = computed(() => {
    const breadcrumbs: Breadcrumb[] = [];
    let currentRoute = this.route.root;

    while (currentRoute.firstChild) {
      currentRoute = currentRoute.firstChild;
      const title = currentRoute.snapshot.data['title'];
      const url = currentRoute.snapshot.url.map(s => s.path).join('/');
      if (title) {
        breadcrumbs.push({ title, url });
      }
    }

    return breadcrumbs;
  });
}
```

## Summary

| Concept | Description |
|---------|-------------|
| `provideRouter` | Configure router in app config |
| `Routes` | Array of route definitions |
| `<router-outlet>` | Placeholder for routed components |
| `routerLink` | Directive for navigation links |
| `Router.navigate()` | Programmatic navigation |
| `ActivatedRoute` | Access route parameters and data |
| Guards | Protect and control route access |
| Lazy Loading | Load routes on demand |
| Resolvers | Pre-fetch data before activation |

## Next Steps

- [Reactive Forms](./09-reactive-forms.md) - Build complex forms
- [HTTP Client](./11-http-client.md) - Make API requests

---

[Previous: Services & Dependency Injection](./07-services-and-dependency-injection.md) | [Back to Index](./README.md) | [Next: Reactive Forms](./09-reactive-forms.md)
