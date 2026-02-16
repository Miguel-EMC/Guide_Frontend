# Testing

Angular v21 introduces Vitest as the new default test runner, providing faster, more modern testing experience. Legacy Jasmine/Karma support remains available for migration. Angular supports Playwright/Cypress for end-to-end tests.

## Testing Setup

### Vitest (Default in Angular 21)

Angular 21 projects use Vitest by default:

```bash
# Run unit tests with Vitest
ng test

# Run with code coverage
ng test --code-coverage

# Run once (CI mode)
ng test --watch=false --run

# Run specific test file
ng test counter.component.spec.ts
```

### Jasmine/Karma (Legacy)

For projects still using Jasmine/Karma:

```bash
# Run unit tests
ng test --test-runner karma

# Run with code coverage
ng test --test-runner karma --code-coverage

# Run once (CI mode)
ng test --test-runner karma --watch=false --browsers=ChromeHeadless
```

## Unit Testing Components

### Basic Component Test (Vitest)

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';

describe('CounterComponent', () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent]  // Standalone component
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should start with count 0', () => {
    expect(component.count()).toBe(0);
  });

  it('should increment count', () => {
    component.increment();
    expect(component.count()).toBe(1);
  });

  it('should decrement count', () => {
    component.increment();
    component.increment();
    component.decrement();
    expect(component.count()).toBe(1);
  });
});
```

### Vitest Advantages

| Feature | Vitest | Jasmine/Karma |
|---------|--------|---------------|
| **Speed** | 5-10x faster | Slower startup |
| **Watch Mode** | Intelligent file watching | Full reload |
| **TypeScript** | Native support | Requires transpilation |
| **API** | Jest-compatible | Jasmine API |
| **Parallel** | Built-in parallel runs | Limited parallelization |
| **Coverage** | v8/c8 built-in | Istanbul coverage |

### Vitest Configuration

Angular 21 automatically configures Vitest. Custom configuration in `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';
import { AngularViteConfig } from '@angular/build';

export default defineConfig(AngularViteConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{ts,js}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html']
    }
  }
}));
```

### Testing DOM

```typescript
describe('CounterComponent DOM', () => {
  let fixture: ComponentFixture<CounterComponent>;
  let element: HTMLElement;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    element = fixture.nativeElement;
    fixture.detectChanges();
  });

  it('should display count', () => {
    const countEl = element.querySelector('.count');
    expect(countEl?.textContent).toContain('0');
  });

  it('should update display on increment', () => {
    const button = element.querySelector<HTMLButtonElement>('.increment');
    button?.click();
    fixture.detectChanges();

    const countEl = element.querySelector('.count');
    expect(countEl?.textContent).toContain('1');
  });

  it('should have increment button', () => {
    const button = element.querySelector('.increment');
    expect(button).toBeTruthy();
    expect(button?.textContent).toContain('+');
  });
});
```

### Testing with Inputs

```typescript
import { Component, input } from '@angular/core';
@Component({
  selector: 'app-greeting',
  template: `<h1>Hello, {{ name() }}!</h1>`
})
export class GreetingComponent {
  name = input('World');
}

describe('GreetingComponent', () => {
  it('should use default name', () => {
    const fixture = TestBed.createComponent(GreetingComponent);
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('Hello, World!');
  });

  it('should use provided name', () => {
    const fixture = TestBed.createComponent(GreetingComponent);
    fixture.componentRef.setInput('name', 'Angular');
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('Hello, Angular!');
  });
});
```

### Testing with Outputs

```typescript
describe('ButtonComponent outputs', () => {
  it('should emit on click', () => {
    const fixture = TestBed.createComponent(ButtonComponent);
    const component = fixture.componentInstance;
    fixture.detectChanges();

    // Spy on output
    const clickSpy = jasmine.createSpy('click');
    component.clicked.subscribe(clickSpy);

    // Trigger click
    const button = fixture.nativeElement.querySelector('button');
    button.click();

    expect(clickSpy).toHaveBeenCalled();
  });
});
```

## Testing Services

### Basic Service Test

```typescript
import { inject } from '@angular/core';
import { TestBed } from '@angular/core/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [UserService]
    });
    service = TestBed.inject(UserService);
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should return empty users initially', () => {
    expect(service.getUsers()()).toEqual([]);
  });

  it('should add user', () => {
    service.addUser({ id: 1, name: 'John' });
    expect(service.getUsers()().length).toBe(1);
  });
});
```

### Vitest Mocking with vi.fn()

```typescript
import { inject } from '@angular/core';
import { vi } from 'vitest';
import { HttpClient } from '@angular/common/http';

describe('UserService with Vitest mocking', () => {
  let service: UserService;
  let mockHttpClient: any;

  beforeEach(() => {
    mockHttpClient = {
      get: vi.fn(),
      post: vi.fn()
    };

    TestBed.configureTestingModule({
      providers: [
        UserService,
        { provide: HttpClient, useValue: mockHttpClient }
      ]
    });
    service = TestBed.inject(UserService);
  });

  it('should fetch users via HTTP', async () => {
    const mockUsers = [{ id: 1, name: 'John' }];
    mockHttpClient.get.mockResolvedValue(mockUsers);

    const users = await service.fetchUsers();
    expect(users).toEqual(mockUsers);
    expect(mockHttpClient.get).toHaveBeenCalledWith('/api/users');
  });

  it('should handle HTTP errors', async () => {
    mockHttpClient.get.mockRejectedValue(new Error('Network error'));

    await expect(service.fetchUsers()).rejects.toThrow('Network error');
  });
});
```

### Testing HTTP Services

```typescript
import { inject } from '@angular/core';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';

describe('ApiService', () => {
  let service: ApiService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [ApiService]
    });

    service = TestBed.inject(ApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();  // Ensure no outstanding requests
  });

  it('should fetch users', () => {
    const mockUsers = [
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  it('should handle error', () => {
    service.getUsers().subscribe({
      error: (error) => {
        expect(error.status).toBe(500);
      }
    });

    const req = httpMock.expectOne('/api/users');
    req.flush('Server Error', { status: 500, statusText: 'Internal Server Error' });
  });

  it('should send POST request', () => {
    const newUser = { name: 'John' };

    service.createUser(newUser).subscribe(user => {
      expect(user.id).toBe(1);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(newUser);
    req.flush({ id: 1, ...newUser });
  });
});
```

### Mocking Services

```typescript
import { signal } from '@angular/core';
describe('UserListComponent with mock', () => {
  let mockUserService: jasmine.SpyObj<UserService>;

  beforeEach(async () => {
    mockUserService = jasmine.createSpyObj('UserService', ['getUsers', 'deleteUser']);
    mockUserService.getUsers.and.returnValue(signal([
      { id: 1, name: 'John' },
      { id: 2, name: 'Jane' }
    ]));

    await TestBed.configureTestingModule({
      imports: [UserListComponent],
      providers: [
        { provide: UserService, useValue: mockUserService }
      ]
    }).compileComponents();
  });

  it('should display users from service', () => {
    const fixture = TestBed.createComponent(UserListComponent);
    fixture.detectChanges();

    const items = fixture.nativeElement.querySelectorAll('.user-item');
    expect(items.length).toBe(2);
  });

  it('should call deleteUser on delete', () => {
    const fixture = TestBed.createComponent(UserListComponent);
    fixture.detectChanges();

    const deleteButton = fixture.nativeElement.querySelector('.delete-btn');
    deleteButton.click();

    expect(mockUserService.deleteUser).toHaveBeenCalledWith(1);
  });
});
```

## Testing with Dependencies

### Testing with Router

```typescript
import { provideRouter } from '@angular/router';
import { RouterTestingHarness } from '@angular/router/testing';

describe('NavigationComponent', () => {
  it('should navigate to users', async () => {
    TestBed.configureTestingModule({
      providers: [
        provideRouter([
          { path: 'users', component: UsersComponent }
        ])
      ]
    });

    const harness = await RouterTestingHarness.create();
    await harness.navigateByUrl('/users');

    expect(harness.routeNativeElement?.textContent).toContain('Users');
  });
});
```

### Testing with Forms

```typescript
import { ReactiveFormsModule } from '@angular/forms';

describe('LoginFormComponent', () => {
  let fixture: ComponentFixture<LoginFormComponent>;
  let component: LoginFormComponent;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [LoginFormComponent, ReactiveFormsModule]
    }).compileComponents();

    fixture = TestBed.createComponent(LoginFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should be invalid when empty', () => {
    expect(component.form.valid).toBeFalse();
  });

  it('should be valid with correct data', () => {
    component.form.patchValue({
      email: 'test@example.com',
      password: 'password123'
    });
    expect(component.form.valid).toBeTrue();
  });

  it('should show email error', () => {
    const emailControl = component.form.get('email');
    emailControl?.setValue('invalid');
    emailControl?.markAsTouched();
    fixture.detectChanges();

    const error = fixture.nativeElement.querySelector('.email-error');
    expect(error).toBeTruthy();
  });

  it('should submit form', () => {
    spyOn(component, 'onSubmit');

    component.form.patchValue({
      email: 'test@example.com',
      password: 'password123'
    });
    fixture.detectChanges();

    const form = fixture.nativeElement.querySelector('form');
    form.dispatchEvent(new Event('submit'));

    expect(component.onSubmit).toHaveBeenCalled();
  });
});
```

## Async Testing

### fakeAsync and tick

```typescript
import { fakeAsync, tick, flush } from '@angular/core/testing';

describe('AsyncComponent', () => {
  it('should load data after delay', fakeAsync(() => {
    const fixture = TestBed.createComponent(AsyncComponent);
    fixture.detectChanges();

    expect(fixture.componentInstance.data()).toBeNull();

    tick(1000);  // Advance time
    fixture.detectChanges();

    expect(fixture.componentInstance.data()).toBeTruthy();
  }));

  it('should complete all async operations', fakeAsync(() => {
    const fixture = TestBed.createComponent(AsyncComponent);
    fixture.componentInstance.startMultipleTimers();

    flush();  // Complete all pending async operations
    fixture.detectChanges();

    expect(fixture.componentInstance.completed()).toBeTrue();
  }));
});
```

### waitForAsync

```typescript
import { waitForAsync } from '@angular/core/testing';

describe('AsyncComponent', () => {
  it('should fetch data', waitForAsync(() => {
    const fixture = TestBed.createComponent(AsyncComponent);
    fixture.detectChanges();

    fixture.whenStable().then(() => {
      fixture.detectChanges();
      expect(fixture.componentInstance.data()).toBeTruthy();
    });
  }));
});
```

## Testing Signals

```typescript
describe('Signal-based component', () => {
  it('should update computed value', () => {
    const fixture = TestBed.createComponent(CalculatorComponent);
    const component = fixture.componentInstance;

    component.a.set(5);
    component.b.set(3);

    expect(component.sum()).toBe(8);
    expect(component.product()).toBe(15);
  });

  it('should react to signal changes in template', () => {
    const fixture = TestBed.createComponent(CounterComponent);
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('0');

    fixture.componentInstance.count.set(5);
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('5');
  });
});
```

## Testing Pipes

```typescript
describe('TruncatePipe', () => {
  let pipe: TruncatePipe;

  beforeEach(() => {
    pipe = new TruncatePipe();
  });

  it('should return empty string for null', () => {
    expect(pipe.transform(null as any)).toBe('');
  });

  it('should not truncate short text', () => {
    expect(pipe.transform('Hello', 10)).toBe('Hello');
  });

  it('should truncate long text', () => {
    expect(pipe.transform('Hello World', 5)).toBe('Hello...');
  });

  it('should use custom ellipsis', () => {
    expect(pipe.transform('Hello World', 5, '---')).toBe('Hello---');
  });
});
```

## Testing Directives

```typescript
import { Component } from '@angular/core';
@Component({
  template: `<div appHighlight [color]="color">Test</div>`
})
class TestHostComponent {
  color = 'yellow';
}

describe('HighlightDirective', () => {
  let fixture: ComponentFixture<TestHostComponent>;
  let element: HTMLElement;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [HighlightDirective],
      declarations: [TestHostComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(TestHostComponent);
    element = fixture.nativeElement.querySelector('div');
    fixture.detectChanges();
  });

  it('should highlight element', () => {
    expect(element.style.backgroundColor).toBe('yellow');
  });

  it('should update on input change', () => {
    fixture.componentInstance.color = 'blue';
    fixture.detectChanges();

    expect(element.style.backgroundColor).toBe('blue');
  });
});
```

## E2E Testing with Playwright

### Setup

```bash
ng add @playwright/test
```

### Basic E2E Test

```typescript
// e2e/app.spec.ts
import { test, expect } from '@playwright/test';

test.describe('App', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should display title', async ({ page }) => {
    await expect(page.locator('h1')).toContainText('Welcome');
  });

  test('should navigate to about', async ({ page }) => {
    await page.click('a[href="/about"]');
    await expect(page).toHaveURL('/about');
  });

  test('should submit form', async ({ page }) => {
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button[type="submit"]');

    await expect(page.locator('.success')).toBeVisible();
  });

  test('should filter list', async ({ page }) => {
    await page.fill('input[name="search"]', 'angular');
    await expect(page.locator('.item')).toHaveCount(1);
  });
});
```

## Migration to Vitest

### Automatic Migration

Angular 21 provides automatic migration from Jasmine to Vitest:

```bash
# Run the migration schematic
ng g @schematics/angular:refactor-jasmine-vitest

# This will:
# - Update test files to Vitest syntax
# - Add Vitest configuration
# - Update package.json dependencies
```

### Manual Migration Steps

1. **Update test imports:**
```typescript
// Before (Jasmine)
import { TestBed } from '@angular/core/testing';
```

```typescript
// After (Vitest)
import { describe, it, expect, beforeEach } from 'vitest';
import { TestBed } from '@angular/core/testing';
```

2. **Replace Jasmine spies with vi.fn():**
```typescript
// Before (Jasmine)
const spy = jasmine.createSpyObj('Service', ['method']);
spy.method.and.returnValue('value');

// After (Vitest)
const spy = { method: vi.fn() };
spy.method.mockReturnValue('value');
```

3. **Update assertions:**
```typescript
// Before (Jasmine)
expect(spy.method).toHaveBeenCalled();
expect(spy.method).toHaveBeenCalledWith('arg');

// After (Vitest) - same API
expect(spy.method).toHaveBeenCalled();
expect(spy.method).toHaveBeenCalledWith('arg');
```

## Test Configuration

### Vitest Configuration (Angular 21 Default)

Created automatically in `vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config';
import { AngularViteConfig } from '@angular/build';

export default defineConfig(AngularViteConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{ts,js}'],
    exclude: ['node_modules', 'dist'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [...],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    },
    watch: true,
    reporters: ['verbose']
  }
}));
```

### Legacy Karma Configuration

For projects still using Karma:

```javascript
// karma.conf.js
module.exports = function (config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-coverage'),
      require('@angular-devkit/build-angular/plugins/karma')
    ],
    coverageReporter: {
      dir: require('path').join(__dirname, './coverage'),
      subdir: '.',
      reporters: [
        { type: 'html' },
        { type: 'text-summary' },
        { type: 'lcov' }
      ]
    },
    reporters: ['progress', 'coverage'],
    browsers: ['Chrome'],
    singleRun: false
  });
};
```

## Summary

| Test Type | Purpose | Tools |
|-----------|---------|-------|
| Unit | Test components, services, pipes | Vitest (default), TestBed |
| Integration | Test component interactions | Vitest, ComponentFixture |
| E2E | Test full user flows | Playwright, Cypress |

| Utility | Purpose |
|---------|---------|
| `TestBed` | Configure testing module |
| `ComponentFixture` | Access component and DOM |
| `fakeAsync/tick` | Control async timing |
| `HttpTestingController` | Mock HTTP requests |
| `vi.fn()` | Vitest function mocking |
| `describe/it/expect` | Vitest test framework |

### Testing in Angular 21

- **Vitest** is now the default test runner for new projects
- **Migration tools** available to convert existing Jasmine tests
- **Better performance** with faster startup and watch mode
- **Modern API** compatible with Jest ecosystem
- **Legacy support** for Jasmine/Karma during migration period

## Next Steps

- [SSR & Hydration](./23-ssr-hydration.md) - Server-side rendering
- [Deployment](./29-deployment.md) - Deploy your application

---

[Previous: Animations](./21-animations.md) | [Back to Index](./README.md) | [Next: SSR & Hydration](./23-ssr-hydration.md)
