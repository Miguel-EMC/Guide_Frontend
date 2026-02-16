# Architecture & Patterns

This guide covers best practices for structuring large-scale Angular applications.

## Project Structure

### Feature-Based Structure

```
src/
├── app/
│   ├── core/                    # Singleton services, guards
│   │   ├── auth/
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.guard.ts
│   │   │   └── auth.interceptor.ts
│   │   ├── http/
│   │   │   └── error.interceptor.ts
│   │   └── core.providers.ts
│   │
│   ├── shared/                  # Reusable components, pipes, directives
│   │   ├── components/
│   │   │   ├── button/
│   │   │   ├── modal/
│   │   │   └── table/
│   │   ├── pipes/
│   │   ├── directives/
│   │   └── index.ts
│   │
│   ├── features/                # Feature modules
│   │   ├── dashboard/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── dashboard.component.ts
│   │   │   └── dashboard.routes.ts
│   │   ├── users/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   ├── models/
│   │   │   └── users.routes.ts
│   │   └── products/
│   │
│   ├── layouts/                 # Layout components
│   │   ├── main-layout/
│   │   └── auth-layout/
│   │
│   ├── app.component.ts
│   ├── app.config.ts
│   └── app.routes.ts
├── environments/
└── styles/
```

### Barrel Exports

**shared/index.ts:**

```typescript
// Components
export * from './components/button/button.component';
export * from './components/modal/modal.component';
export * from './components/table/table.component';

// Pipes
export * from './pipes/truncate.pipe';
export * from './pipes/date-format.pipe';

// Directives
export * from './directives/tooltip.directive';
```

**Usage:**

```typescript
import { ButtonComponent, ModalComponent, TruncatePipe } from '@shared';
```

### Path Aliases

**tsconfig.json:**

```json
{
  "compilerOptions": {
    "paths": {
      "@app/*": ["src/app/*"],
      "@core/*": ["src/app/core/*"],
      "@shared/*": ["src/app/shared/*"],
      "@features/*": ["src/app/features/*"],
      "@env/*": ["src/environments/*"]
    }
  }
}
```

## Component Patterns

### Smart vs Presentational

**Smart Component (Container):**

```typescript
import { Component, inject } from '@angular/core';
@Component({
  selector: 'app-user-list-container',
  standalone: true,
  imports: [UserListComponent],
  template: `
    <app-user-list
      [users]="users()"
      [loading]="loading()"
      (userSelected)="onUserSelected($event)"
      (userDeleted)="onUserDeleted($event)"
    />
  `
})
export class UserListContainerComponent {
  private userService = inject(UserService);

  users = this.userService.users;
  loading = this.userService.loading;

  onUserSelected(user: User) {
    this.router.navigate(['/users', user.id]);
  }

  onUserDeleted(user: User) {
    this.userService.delete(user.id);
  }
}
```

**Presentational Component:**

```typescript
import { Component, input, ChangeDetectionStrategy, output } from '@angular/core';
@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    @if (loading()) {
      <app-spinner />
    } @else {
      @for (user of users(); track user.id) {
        <app-user-card
          [user]="user"
          (click)="userSelected.emit(user)"
        >
          <button (click)="userDeleted.emit(user); $event.stopPropagation()">
            Delete
          </button>
        </app-user-card>
      }
    }
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent {
  users = input.required<User[]>();
  loading = input(false);

  userSelected = output<User>();
  userDeleted = output<User>();
}
```

### Component Composition

```typescript
import { Component, input } from '@angular/core';
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="card" [class.elevated]="elevated()">
      <header class="card-header">
        <ng-content select="[card-header]" />
      </header>
      <div class="card-body">
        <ng-content />
      </div>
      <footer class="card-footer">
        <ng-content select="[card-footer]" />
      </footer>
    </div>
  `
})
export class CardComponent {
  elevated = input(false);
}

// Usage
@Component({
  template: `
    <app-card [elevated]="true">
      <h3 card-header>Title</h3>
      <p>Content goes here</p>
      <button card-footer>Action</button>
    </app-card>
  `
})
```

## Service Patterns

### Repository Pattern

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';
// Base repository
export abstract class BaseRepository<T extends { id: number }> {
  protected http = inject(HttpClient);
  protected abstract baseUrl: string;

  getAll(): Observable<T[]> {
    return this.http.get<T[]>(this.baseUrl);
  }

  getById(id: number): Observable<T> {
    return this.http.get<T>(`${this.baseUrl}/${id}`);
  }

  create(entity: Omit<T, 'id'>): Observable<T> {
    return this.http.post<T>(this.baseUrl, entity);
  }

  update(id: number, entity: Partial<T>): Observable<T> {
    return this.http.put<T>(`${this.baseUrl}/${id}`, entity);
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }
}

// Concrete implementation
@Injectable({ providedIn: 'root' })
export class UserRepository extends BaseRepository<User> {
  protected baseUrl = '/api/users';

  // Additional user-specific methods
  getByEmail(email: string): Observable<User> {
    return this.http.get<User>(`${this.baseUrl}`, {
      params: { email }
    });
  }
}
```

### Facade Pattern

```typescript
import { Injectable, inject, computed } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class ProductFacade {
  private productService = inject(ProductService);
  private cartService = inject(CartService);
  private notificationService = inject(NotificationService);

  // Expose state
  readonly products = this.productService.products;
  readonly loading = this.productService.loading;
  readonly cart = this.cartService.items;

  // Computed
  readonly featuredProducts = computed(() =>
    this.products().filter(p => p.featured)
  );

  // Actions
  async loadProducts() {
    try {
      await this.productService.load();
    } catch (error) {
      this.notificationService.error('Failed to load products');
    }
  }

  addToCart(product: Product, quantity: number) {
    this.cartService.add(product, quantity);
    this.notificationService.success('Added to cart');
  }

  async checkout() {
    try {
      await this.cartService.checkout();
      this.notificationService.success('Order placed!');
    } catch (error) {
      this.notificationService.error('Checkout failed');
    }
  }
}
```

## State Management Patterns

### Simple Signal Store

```typescript
import { Injectable, signal, computed } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class TodoStore {
  // Private mutable state
  private _todos = signal<Todo[]>([]);
  private _filter = signal<TodoFilter>('all');
  private _loading = signal(false);

  // Public read-only state
  readonly todos = this._todos.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly loading = this._loading.asReadonly();

  // Computed
  readonly filteredTodos = computed(() => {
    const todos = this._todos();
    switch (this._filter()) {
      case 'active': return todos.filter(t => !t.completed);
      case 'completed': return todos.filter(t => t.completed);
      default: return todos;
    }
  });

  readonly stats = computed(() => ({
    total: this._todos().length,
    active: this._todos().filter(t => !t.completed).length,
    completed: this._todos().filter(t => t.completed).length
  }));

  // Actions
  addTodo(title: string) {
    const todo: Todo = { id: Date.now(), title, completed: false };
    this._todos.update(todos => [...todos, todo]);
  }

  toggleTodo(id: number) {
    this._todos.update(todos =>
      todos.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );
  }

  deleteTodo(id: number) {
    this._todos.update(todos => todos.filter(t => t.id !== id));
  }

  setFilter(filter: TodoFilter) {
    this._filter.set(filter);
  }
}
```

## Routing Patterns

### Route Guards

```typescript
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
// auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

// role.guard.ts
export const roleGuard = (allowedRoles: string[]): CanActivateFn => {
  return (route, state) => {
    const auth = inject(AuthService);
    const role = auth.currentUser()?.role;

    if (role && allowedRoles.includes(role)) {
      return true;
    }

    return inject(Router).createUrlTree(['/unauthorized']);
  };
};

// Usage
{
  path: 'admin',
  canActivate: [authGuard, roleGuard(['admin'])]
}
```

### Lazy Loading

```typescript
import { Routes } from '@angular/router';
export const routes: Routes = [
  {
    path: '',
    loadComponent: () => import('./home/home.component').then(m => m.HomeComponent)
  },
  {
    path: 'users',
    loadChildren: () => import('./features/users/users.routes').then(m => m.USERS_ROUTES)
  },
  {
    path: 'admin',
    canMatch: [authGuard, roleGuard(['admin'])],
    loadChildren: () => import('./features/admin/admin.routes').then(m => m.ADMIN_ROUTES)
  }
];
```

## Error Handling

### Global Error Handler

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpErrorResponse } from '@angular/common/http';
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  private notificationService = inject(NotificationService);
  private loggingService = inject(LoggingService);

  handleError(error: Error): void {
    // Log error
    this.loggingService.logError(error);

    // Show user-friendly message
    const message = this.getErrorMessage(error);
    this.notificationService.error(message);

    // Rethrow for debugging
    console.error(error);
  }

  private getErrorMessage(error: Error): string {
    if (error instanceof HttpErrorResponse) {
      return this.getHttpErrorMessage(error);
    }
    return 'An unexpected error occurred';
  }

  private getHttpErrorMessage(error: HttpErrorResponse): string {
    switch (error.status) {
      case 400: return 'Invalid request';
      case 401: return 'Please log in';
      case 403: return 'Access denied';
      case 404: return 'Not found';
      case 500: return 'Server error';
      default: return 'Connection error';
    }
  }
}

// Provide in app.config.ts
{ provide: ErrorHandler, useClass: GlobalErrorHandler }
```

## Testing Architecture

### Test Structure

```
src/app/features/users/
├── components/
│   ├── user-list/
│   │   ├── user-list.component.ts
│   │   └── user-list.component.spec.ts
│   └── user-form/
│       ├── user-form.component.ts
│       └── user-form.component.spec.ts
├── services/
│   ├── user.service.ts
│   └── user.service.spec.ts
└── testing/
    ├── user.mock.ts
    └── user.service.mock.ts
```

### Test Utilities

```typescript
// testing/test-utils.ts
export function createMockService<T>(service: Type<T>): jasmine.SpyObj<T> {
  const methods = Object.getOwnPropertyNames(service.prototype)
    .filter(name => name !== 'constructor');
  return jasmine.createSpyObj(service.name, methods);
}

// Usage
const mockUserService = createMockService(UserService);
```

## Performance Patterns

### Change Detection Strategy

```typescript
import { Component, signal, ChangeDetectionStrategy } from '@angular/core';
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `...`
})
export class OptimizedComponent {
  // Use signals for automatic change detection
  data = input.required<Data>();
  count = signal(0);
}
```

### Track By

```html
@for (item of items(); track item.id) {
  <app-item [data]="item" />
}
```

### Defer Loading

```html
@defer (on viewport) {
  <app-heavy-component />
} @placeholder {
  <div class="skeleton"></div>
}
```

## Summary

| Pattern | Use Case |
|---------|----------|
| Feature Modules | Organize by business domain |
| Smart/Presentational | Separate logic from UI |
| Repository | Abstract data access |
| Facade | Simplify complex subsystems |
| Signal Store | Lightweight state management |
| Guards | Route protection |
| Error Handler | Centralized error handling |

## Next Steps

- [Deployment](./29-deployment.md) - Deploy your application
- [Project: Task Manager App](./30-project-task-manager.md) - Build a complete app

---

[Previous: PWA & Service Workers](./27-pwa.md) | [Back to Index](./README.md) | [Next: Deployment](./29-deployment.md)
