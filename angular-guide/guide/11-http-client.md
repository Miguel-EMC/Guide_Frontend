# HTTP Client

Angular's HttpClient provides a powerful API for making HTTP requests to communicate with backend services.

## Setup

**app.config.ts:**

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors, withFetch } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withFetch(),           // Use fetch API (recommended)
      withInterceptors([])   // Add interceptors
    )
  ]
};
```

## Basic Requests

### GET Request

```typescript
import { Component, inject, signal } from '@angular/core';
import { HttpClient } from '@angular/common/http';

interface User {
  id: number;
  name: string;
  email: string;
}

@Component({
  standalone: true,
  template: `
    @if (loading()) {
      <p>Loading...</p>
    }

    @for (user of users(); track user.id) {
      <div>{{ user.name }} - {{ user.email }}</div>
    }
  `
})
export class UsersComponent {
  private http = inject(HttpClient);

  users = signal<User[]>([]);
  loading = signal(false);

  constructor() {
    this.loadUsers();
  }

  loadUsers() {
    this.loading.set(true);

    this.http.get<User[]>('https://api.example.com/users')
      .subscribe({
        next: (users) => this.users.set(users),
        error: (error) => console.error('Error:', error),
        complete: () => this.loading.set(false)
      });
  }
}
```

### POST Request

```typescript
createUser(user: CreateUserDto) {
  return this.http.post<User>('https://api.example.com/users', user);
}

// Usage
this.createUser({ name: 'John', email: 'john@example.com' })
  .subscribe(newUser => {
    console.log('Created:', newUser);
  });
```

### PUT Request

```typescript
updateUser(id: number, user: UpdateUserDto) {
  return this.http.put<User>(`https://api.example.com/users/${id}`, user);
}
```

### PATCH Request

```typescript
patchUser(id: number, updates: Partial<User>) {
  return this.http.patch<User>(`https://api.example.com/users/${id}`, updates);
}
```

### DELETE Request

```typescript
deleteUser(id: number) {
  return this.http.delete<void>(`https://api.example.com/users/${id}`);
}
```

## Request Options

### Headers

```typescript
import { HttpHeaders } from '@angular/common/http';

const headers = new HttpHeaders()
  .set('Content-Type', 'application/json')
  .set('Authorization', 'Bearer token123');

this.http.get<User[]>('/api/users', { headers });

// Or inline
this.http.get<User[]>('/api/users', {
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token123'
  }
});
```

### Query Parameters

```typescript
import { HttpParams } from '@angular/common/http';

const params = new HttpParams()
  .set('page', '1')
  .set('limit', '10')
  .set('sort', 'name');

this.http.get<User[]>('/api/users', { params });

// Or inline
this.http.get<User[]>('/api/users', {
  params: { page: '1', limit: '10', sort: 'name' }
});

// Multiple values
const params = new HttpParams()
  .appendAll({ tags: ['angular', 'typescript'] });
```

### Response Types

```typescript
// JSON (default)
this.http.get<User>('/api/user');

// Text
this.http.get('/api/text', { responseType: 'text' });

// Blob (file download)
this.http.get('/api/file', { responseType: 'blob' });

// ArrayBuffer
this.http.get('/api/binary', { responseType: 'arraybuffer' });

// Full response (includes headers, status)
this.http.get<User>('/api/user', { observe: 'response' })
  .subscribe(response => {
    console.log('Status:', response.status);
    console.log('Headers:', response.headers);
    console.log('Body:', response.body);
  });

// Events (for progress)
this.http.get('/api/large-file', {
  observe: 'events',
  reportProgress: true
});
```

## HTTP Interceptors

### Functional Interceptors (Recommended)

**auth.interceptor.ts:**

```typescript
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(req);
};
```

**logging.interceptor.ts:**

```typescript
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const started = Date.now();

  return next(req).pipe(
    tap({
      next: (event) => {
        if (event instanceof HttpResponse) {
          const elapsed = Date.now() - started;
          console.log(`${req.method} ${req.urlWithParams} - ${elapsed}ms`);
        }
      },
      error: (error) => {
        const elapsed = Date.now() - started;
        console.error(`${req.method} ${req.urlWithParams} failed - ${elapsed}ms`, error);
      }
    })
  );
};
```

**error.interceptor.ts:**

```typescript
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const notificationService = inject(NotificationService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      let message = 'An error occurred';

      switch (error.status) {
        case 401:
          message = 'Please log in';
          router.navigate(['/login']);
          break;
        case 403:
          message = 'Access denied';
          break;
        case 404:
          message = 'Resource not found';
          break;
        case 500:
          message = 'Server error';
          break;
      }

      notificationService.error(message);
      return throwError(() => error);
    })
  );
};
```

### Register Interceptors

```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        loggingInterceptor,
        errorInterceptor
      ])
    )
  ]
};
```

### Retry Interceptor

```typescript
import { retry, timer } from 'rxjs';

export const retryInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    retry({
      count: 3,
      delay: (error, retryCount) => {
        // Only retry on server errors
        if (error.status >= 500) {
          return timer(1000 * retryCount); // Exponential backoff
        }
        throw error;
      }
    })
  );
};
```

### Cache Interceptor

```typescript
const cache = new Map<string, HttpResponse<any>>();

export const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  // Only cache GET requests
  if (req.method !== 'GET') {
    return next(req);
  }

  const cached = cache.get(req.urlWithParams);
  if (cached) {
    return of(cached.clone());
  }

  return next(req).pipe(
    tap(event => {
      if (event instanceof HttpResponse) {
        cache.set(req.urlWithParams, event.clone());
      }
    })
  );
};
```

## Data Service Pattern

```typescript
@Injectable({ providedIn: 'root' })
export class ProductService {
  private http = inject(HttpClient);
  private apiUrl = inject(API_URL);

  // State
  private products = signal<Product[]>([]);
  private loading = signal(false);
  private error = signal<string | null>(null);

  // Selectors
  readonly products$ = this.products.asReadonly();
  readonly loading$ = this.loading.asReadonly();
  readonly error$ = this.error.asReadonly();

  getAll(): Observable<Product[]> {
    this.loading.set(true);
    this.error.set(null);

    return this.http.get<Product[]>(`${this.apiUrl}/products`).pipe(
      tap(products => this.products.set(products)),
      catchError(err => {
        this.error.set('Failed to load products');
        return throwError(() => err);
      }),
      finalize(() => this.loading.set(false))
    );
  }

  getById(id: number): Observable<Product> {
    return this.http.get<Product>(`${this.apiUrl}/products/${id}`);
  }

  create(data: CreateProductDto): Observable<Product> {
    return this.http.post<Product>(`${this.apiUrl}/products`, data).pipe(
      tap(product => {
        this.products.update(products => [...products, product]);
      })
    );
  }

  update(id: number, data: UpdateProductDto): Observable<Product> {
    return this.http.put<Product>(`${this.apiUrl}/products/${id}`, data).pipe(
      tap(updated => {
        this.products.update(products =>
          products.map(p => p.id === id ? updated : p)
        );
      })
    );
  }

  delete(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/products/${id}`).pipe(
      tap(() => {
        this.products.update(products =>
          products.filter(p => p.id !== id)
        );
      })
    );
  }
}
```

## File Upload

### Single File

```typescript
uploadFile(file: File): Observable<UploadResponse> {
  const formData = new FormData();
  formData.append('file', file, file.name);

  return this.http.post<UploadResponse>('/api/upload', formData);
}
```

### With Progress

```typescript
uploadWithProgress(file: File): Observable<number | UploadResponse> {
  const formData = new FormData();
  formData.append('file', file);

  return this.http.post<UploadResponse>('/api/upload', formData, {
    reportProgress: true,
    observe: 'events'
  }).pipe(
    map(event => {
      if (event.type === HttpEventType.UploadProgress) {
        return Math.round(100 * event.loaded / (event.total || 1));
      }
      if (event.type === HttpEventType.Response) {
        return event.body as UploadResponse;
      }
      return 0;
    }),
    filter(value => typeof value === 'number' || value !== 0)
  );
}
```

### Component Usage

```typescript
@Component({
  template: `
    <input type="file" (change)="onFileSelected($event)" />

    @if (uploadProgress() > 0) {
      <div class="progress">
        <div [style.width.%]="uploadProgress()"></div>
      </div>
      <p>{{ uploadProgress() }}%</p>
    }
  `
})
export class UploadComponent {
  private uploadService = inject(UploadService);

  uploadProgress = signal(0);

  onFileSelected(event: Event) {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (!file) return;

    this.uploadService.uploadWithProgress(file).subscribe({
      next: (result) => {
        if (typeof result === 'number') {
          this.uploadProgress.set(result);
        } else {
          console.log('Upload complete:', result);
        }
      },
      error: (err) => console.error('Upload failed:', err)
    });
  }
}
```

## File Download

```typescript
downloadFile(url: string, filename: string) {
  this.http.get(url, { responseType: 'blob' })
    .subscribe(blob => {
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = filename;
      link.click();
      URL.revokeObjectURL(link.href);
    });
}
```

## Typed HTTP Client

```typescript
interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
}

interface PaginatedResponse<T> {
  items: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
}

@Injectable({ providedIn: 'root' })
export class ApiService {
  private http = inject(HttpClient);

  get<T>(url: string, params?: HttpParams): Observable<ApiResponse<T>> {
    return this.http.get<ApiResponse<T>>(url, { params });
  }

  getPaginated<T>(
    url: string,
    page: number,
    pageSize: number
  ): Observable<PaginatedResponse<T>> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('pageSize', pageSize.toString());

    return this.http.get<PaginatedResponse<T>>(url, { params });
  }

  post<T, R>(url: string, body: T): Observable<ApiResponse<R>> {
    return this.http.post<ApiResponse<R>>(url, body);
  }
}
```

## Error Handling

### Centralized Error Handler

```typescript
@Injectable({ providedIn: 'root' })
export class HttpErrorHandler {
  handle(error: HttpErrorResponse): Observable<never> {
    let message: string;

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      message = `Client Error: ${error.error.message}`;
    } else {
      // Server-side error
      message = this.getServerErrorMessage(error);
    }

    console.error('HTTP Error:', message);
    return throwError(() => new Error(message));
  }

  private getServerErrorMessage(error: HttpErrorResponse): string {
    switch (error.status) {
      case 400:
        return error.error?.message || 'Bad request';
      case 401:
        return 'Unauthorized';
      case 403:
        return 'Forbidden';
      case 404:
        return 'Not found';
      case 422:
        return this.formatValidationErrors(error.error?.errors);
      case 500:
        return 'Internal server error';
      default:
        return 'Unknown error occurred';
    }
  }

  private formatValidationErrors(errors: Record<string, string[]>): string {
    if (!errors) return 'Validation failed';

    return Object.entries(errors)
      .map(([field, messages]) => `${field}: ${messages.join(', ')}`)
      .join('; ');
  }
}
```

## Testing HTTP

```typescript
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('ProductService', () => {
  let service: ProductService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ProductService]
    });

    service = TestBed.inject(ProductService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify(); // Ensure no outstanding requests
  });

  it('should get products', () => {
    const mockProducts = [{ id: 1, name: 'Product 1' }];

    service.getAll().subscribe(products => {
      expect(products).toEqual(mockProducts);
    });

    const req = httpMock.expectOne('/api/products');
    expect(req.request.method).toBe('GET');
    req.flush(mockProducts);
  });

  it('should handle errors', () => {
    service.getAll().subscribe({
      error: (error) => {
        expect(error.status).toBe(500);
      }
    });

    const req = httpMock.expectOne('/api/products');
    req.flush('Error', { status: 500, statusText: 'Server Error' });
  });
});
```

## Summary

| Concept | Description |
|---------|-------------|
| `HttpClient` | Make HTTP requests |
| `provideHttpClient()` | Configure in app config |
| Interceptors | Modify requests/responses |
| `HttpParams` | Query parameters |
| `HttpHeaders` | Request headers |
| Typed responses | Type-safe API calls |
| Error handling | Centralized error management |

## Next Steps

- [Angular Signals](./12-signals.md) - Reactive state management
- [RxJS & Observables](./13-rxjs.md) - Advanced async patterns

---

[Previous: Signal Forms](./10-signal-forms.md) | [Back to Index](./README.md) | [Next: Angular Signals](./12-signals.md)
