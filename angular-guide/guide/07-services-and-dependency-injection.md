# Services & Dependency Injection

Services are classes that encapsulate business logic, data access, and shared functionality. Dependency Injection (DI) is a design pattern that Angular uses to provide these services to components.

## Creating Services

### Generate Service

```bash
ng generate service user
# or
ng g s user
```

### Basic Service

```typescript
import { Injectable, signal, computed } from '@angular/core';

@Injectable({
  providedIn: 'root'  // Singleton available app-wide
})
export class UserService {
  // State
  private currentUser = signal<User | null>(null);
  private users = signal<User[]>([]);

  // Computed values
  isLoggedIn = computed(() => this.currentUser() !== null);
  userCount = computed(() => this.users().length);

  // Methods
  login(credentials: LoginCredentials): void {
    // Login logic
  }

  logout(): void {
    this.currentUser.set(null);
  }

  getUsers(): User[] {
    return this.users();
  }

  addUser(user: User): void {
    this.users.update(users => [...users, user]);
  }
}
```

## Injecting Services

### inject() Function (Recommended)

```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    @if (userService.isLoggedIn()) {
      <p>Welcome!</p>
    }
    @for (user of userService.getUsers(); track user.id) {
      <p>{{ user.name }}</p>
    }
  `
})
export class UserListComponent {
  userService = inject(UserService);
}
```

### Constructor Injection (Classic)

```typescript
import { Component } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `...`
})
export class UserListComponent {
  constructor(private userService: UserService) {}
}
```

## providedIn Options

### Root Level (Singleton)

```typescript
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'root'  // Single instance for entire app
})
export class ConfigService {}
```

### Platform Level

```typescript
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'platform'  // Shared across multiple apps
})
export class AnalyticsService {}
```

### Any Level

```typescript
import { Injectable } from '@angular/core';
@Injectable({
  providedIn: 'any'  // New instance per lazy-loaded module
})
export class FeatureService {}
```

### Component Level (Per Instance)

```typescript
import { Component, inject } from '@angular/core';
@Component({
  selector: 'app-feature',
  providers: [FeatureService]  // New instance per component
})
export class FeatureComponent {
  featureService = inject(FeatureService);
}
```

## Injection Tokens

### Basic Token

```typescript
import { InjectionToken, inject, Injectable, ApplicationConfig } from '@angular/core';

// Define token
export const API_URL = new InjectionToken<string>('API URL');

// Provide in config
export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_URL, useValue: 'https://api.example.com' }
  ]
};

// Use in service
@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = inject(API_URL);

  getEndpoint(path: string): string {
    return `${this.apiUrl}/${path}`;
  }
}
```

### Factory Token

```typescript
import { inject } from '@angular/core';
export const LOGGER = new InjectionToken<Logger>('Logger', {
  providedIn: 'root',
  factory: () => {
    const isDev = inject(IS_DEV_MODE);
    return isDev ? new ConsoleLogger() : new ProductionLogger();
  }
});
```

## Provider Types

### useClass

```typescript
// Provide a class
{ provide: Logger, useClass: ConsoleLogger }

// Conditional
{ provide: Logger, useClass: environment.production ? ProductionLogger : DebugLogger }
```

### useValue

```typescript
// Provide a value
{ provide: API_URL, useValue: 'https://api.example.com' }

// Provide config object
{ provide: APP_CONFIG, useValue: { theme: 'dark', lang: 'en' } }
```

### useFactory

```typescript
import { HttpClient } from '@angular/common/http';
// Provide with factory function
{
  provide: DataService,
  useFactory: (http: HttpClient, config: AppConfig) => {
    return config.useMock
      ? new MockDataService()
      : new RealDataService(http);
  },
  deps: [HttpClient, APP_CONFIG]
}
```

### useExisting

```typescript
// Alias to existing provider
{ provide: AbstractLogger, useExisting: ConsoleLogger }
```

## Hierarchical Injection

### App Level

```typescript
import { provideHttpClient } from '@angular/common/http';
import { ApplicationConfig } from '@angular/core';
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    { provide: API_URL, useValue: 'https://api.example.com' }
  ]
};
```

### Route Level

```typescript
import { Routes } from '@angular/router';
// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    providers: [AdminService],  // Only available to admin routes
    children: [
      { path: '', component: AdminDashboardComponent }
    ]
  }
];
```

### Component Level

```typescript
import { Component } from '@angular/core';
@Component({
  selector: 'app-feature',
  standalone: true,
  providers: [
    FeatureService,
    { provide: Logger, useClass: FeatureLogger }
  ]
})
export class FeatureComponent {}
```

## Optional and Self/SkipSelf

### Optional Injection

```typescript
import { Component, inject } from '@angular/core';
@Component({
  template: `...`
})
export class OptionalComponent {
  // Won't throw if not provided
  logger = inject(Logger, { optional: true });

  log(message: string) {
    this.logger?.log(message);
  }
}
```

### Self Injection

```typescript
import { Component, inject } from '@angular/core';
@Component({
  providers: [LocalService]
})
export class SelfComponent {
  // Only look in this component's injector
  service = inject(LocalService, { self: true });
}
```

### SkipSelf Injection

```typescript
import { Component, inject } from '@angular/core';
@Component({
  providers: [ConfigService]
})
export class ChildComponent {
  // Skip this component, look in parent
  parentConfig = inject(ConfigService, { skipSelf: true });

  // Get this component's instance
  localConfig = inject(ConfigService, { self: true });
}
```

### Host Injection

```typescript
import { inject, Directive } from '@angular/core';
@Directive({
  selector: '[appChild]'
})
export class ChildDirective {
  // Only look in host component's injector
  hostService = inject(HostService, { host: true });
}
```

## Service Patterns

### State Management Service

```typescript
import { Injectable, computed, signal } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class CartService {
  // Private state
  private items = signal<CartItem[]>([]);

  // Public computed values
  readonly cartItems = this.items.asReadonly();
  readonly itemCount = computed(() => this.items().length);
  readonly total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  readonly isEmpty = computed(() => this.items().length === 0);

  // Actions
  addItem(product: Product, quantity: number = 1): void {
    this.items.update(items => {
      const existing = items.find(i => i.productId === product.id);
      if (existing) {
        return items.map(i =>
          i.productId === product.id
            ? { ...i, quantity: i.quantity + quantity }
            : i
        );
      }
      return [...items, { productId: product.id, product, quantity, price: product.price }];
    });
  }

  removeItem(productId: number): void {
    this.items.update(items => items.filter(i => i.productId !== productId));
  }

  updateQuantity(productId: number, quantity: number): void {
    if (quantity <= 0) {
      this.removeItem(productId);
      return;
    }
    this.items.update(items =>
      items.map(i =>
        i.productId === productId ? { ...i, quantity } : i
      )
    );
  }

  clear(): void {
    this.items.set([]);
  }
}
```

### Data Service with HTTP

```typescript
import { Injectable, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs/operators';
import { Observable, of } from 'rxjs';
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL);

  // Cache
  private productsCache = signal<Product[]>([]);
  private lastFetch = signal<Date | null>(null);

  getProducts(): Observable<Product[]> {
    // Return cache if fresh
    if (this.isCacheFresh()) {
      return of(this.productsCache());
    }

    return this.http.get<Product[]>(`${this.apiUrl}/products`).pipe(
      tap(products => {
        this.productsCache.set(products);
        this.lastFetch.set(new Date());
      })
    );
  }

  getProduct(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/products/${id}`);
  }

  createProduct(product: CreateProductDto): Observable<Product> {
    return this.http.post<Product>(`${this.apiUrl}/products`, product).pipe(
      tap(newProduct => {
        this.productsCache.update(products => [...products, newProduct]);
      })
    );
  }

  updateProduct(id: number, product: UpdateProductDto): Observable<Product> {
    return this.http.put<Product>(`${this.apiUrl}/products/${id}`, product).pipe(
      tap(updated => {
        this.productsCache.update(products =>
          products.map(p => p.id === id ? updated : p)
        );
      })
    );
  }

  deleteProduct(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/products/${id}`).pipe(
      tap(() => {
        this.productsCache.update(products =>
          products.filter(p => p.id !== id)
        );
      })
    );
  }

  private isCacheFresh(): boolean {
    const last = this.lastFetch();
    if (!last) return false;
    const fiveMinutes = 5 * 60 * 1000;
    return Date.now() - last.getTime() < fiveMinutes;
  }
}
```

### Auth Service

```typescript
import { Injectable, inject, computed, signal } from '@angular/core';
import { Router } from '@angular/router';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs/operators';
import { Observable } from 'rxjs';
@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);

  private currentUser = signal<User | null>(null);
  private token = signal<string | null>(null);

  readonly user = this.currentUser.asReadonly();
  readonly isAuthenticated = computed(() => this.token() !== null);
  readonly isAdmin = computed(() => this.currentUser()?.role === 'admin');

  constructor() {
    // Restore from storage
    this.loadFromStorage();
  }

  login(credentials: LoginDto): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.setSession(response);
      })
    );
  }

  logout(): void {
    this.currentUser.set(null);
    this.token.set(null);
    localStorage.removeItem('auth_token');
    localStorage.removeItem('user');
    this.router.navigate(['/login']);
  }

  refreshToken(): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/refresh', {}).pipe(
      tap(response => this.setSession(response))
    );
  }

  getToken(): string | null {
    return this.token();
  }

  private setSession(response: AuthResponse): void {
    this.token.set(response.token);
    this.currentUser.set(response.user);
    localStorage.setItem('auth_token', response.token);
    localStorage.setItem('user', JSON.stringify(response.user));
  }

  private loadFromStorage(): void {
    const token = localStorage.getItem('auth_token');
    const userJson = localStorage.getItem('user');

    if (token && userJson) {
      this.token.set(token);
      this.currentUser.set(JSON.parse(userJson));
    }
  }
}
```

### Notification Service

```typescript
import { Injectable, signal } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class NotificationService {
  private notifications = signal<Notification[]>([]);

  readonly all = this.notifications.asReadonly();

  show(message: string, type: NotificationType = 'info', duration: number = 5000): void {
    const notification: Notification = {
      id: Date.now(),
      message,
      type,
      createdAt: new Date()
    };

    this.notifications.update(n => [...n, notification]);

    if (duration > 0) {
      setTimeout(() => this.dismiss(notification.id), duration);
    }
  }

  success(message: string): void {
    this.show(message, 'success');
  }

  error(message: string): void {
    this.show(message, 'error', 0); // Don't auto-dismiss errors
  }

  warning(message: string): void {
    this.show(message, 'warning');
  }

  dismiss(id: number): void {
    this.notifications.update(n => n.filter(notif => notif.id !== id));
  }

  dismissAll(): void {
    this.notifications.set([]);
  }
}
```

## Testing Services

```typescript
import { inject } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('ProductService', () => {
  let service: ProductService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [
        ProductService,
        { provide: API_URL, useValue: 'https://api.test.com' }
      ]
    });

    service = TestBed.inject(ProductService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch products', () => {
    const mockProducts = [{ id: 1, name: 'Product 1' }];

    service.getProducts().subscribe(products => {
      expect(products).toEqual(mockProducts);
    });

    const req = httpMock.expectOne('https://api.test.com/products');
    expect(req.request.method).toBe('GET');
    req.flush(mockProducts);
  });
});
```

## Summary

| Concept | Description |
|---------|-------------|
| `@Injectable` | Marks class as injectable service |
| `providedIn: 'root'` | Singleton for entire app |
| `inject()` | Function to inject dependencies |
| `InjectionToken` | Token for non-class dependencies |
| `useClass/Value/Factory` | Provider configurations |
| Hierarchical DI | Providers at different levels |
| Optional/Self/SkipSelf | Control injection lookup |

## Next Steps

- [Routing](./08-routing.md) - Navigation between views
- [HTTP Client](./11-http-client.md) - Making HTTP requests

---

[Previous: Pipes](./06-pipes.md) | [Back to Index](./README.md) | [Next: Routing](./08-routing.md)
