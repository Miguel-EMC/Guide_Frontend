# Zoneless Change Detection (v21)

Angular v21 makes **Zoneless change detection** the default for new applications. This represents a fundamental shift in how Angular handles reactivity, providing better performance, smaller bundles, and more predictable behavior.

## Overview

Zoneless applications don't use Zone.js to automatically trigger change detection. Instead, they rely on explicit reactivity through Signals and event handlers.

| Aspect | Zone-Based | Zoneless |
|--------|------------|----------|
| **Bundle Size** | ~40KB (Zone.js) | No Zone.js overhead |
| **Performance** | Automatic but slower | Manual but faster |
| **Predictability** | "Magic" updates | Explicit updates |
| **Debugging** | Complex stack traces | Clear call stacks |
| **Ecosystem** | Better with legacy code | Modern web standards |

## Understanding Zone.js

Zone.js patches browser APIs to automatically detect changes:

```typescript
// Zone.js patches these APIs automatically
setTimeout(() => {
  // Zone.js triggers change detection here
  this.data = 'updated';
});

fetch('/api/data').then(response => {
  // Zone.js triggers change detection here  
  this.data = response.data;
});
```

**Problems with Zone.js:**
- Adds bundle size overhead (~40KB)
- Patches native browser APIs
- Can cause performance issues
- Makes debugging harder
- Interferes with some modern APIs

## Zoneless with Signals

Zoneless applications use Signals for reactive updates:

```typescript
@Component({
  standalone: true,
  template: `
    <h1>{{ title() }}</h1>
    <button (click)="updateTitle()">Update</button>
  `
})
export class MyComponent {
  title = signal('Hello');

  updateTitle() {
    // Signal update automatically triggers change detection
    this.title.set('Hello World');
  }
}
```

## Setup

### New Applications (Default)

Angular 21 applications are zoneless by default:

```typescript
// app.config.ts (Angular 21)
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    // No Zone.js provider - zoneless by default
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

### Existing Applications

Convert to zoneless:

```bash
# Use the migration tool
ng g @schematics/angular:onpush-zoneless-migration

# Or manually remove Zone.js
```

**Manual conversion:**

```typescript
// Before (Zone-based)
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    provideZoneChangeDetection({ eventCoalescing: true }),
    // ... other providers
  ]
};

// After (Zoneless)
import { ApplicationConfig } from '@angular/core';

export const appConfig: ApplicationConfig = {
  providers: [
    // No Zone.js provider needed
    // ... other providers
  ]
};
```

## Signal-Based Reactivity

### Using Signals

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  standalone: true,
  template: `
    <div>
      <p>Count: {{ count() }}</p>
      <p>Doubled: {{ doubled() }}</p>
      <button (click)="increment()">Increment</button>
    </div>
  `
})
export class CounterComponent {
  count = signal(0);
  
  // Computed signals automatically update
  doubled = computed(() => this.count() * 2);

  increment() {
    this.count.update(n => n + 1);
  }

  constructor() {
    // Effects run when dependencies change
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }
}
```

### Model Signals (v21)

```typescript
import { model } from '@angular/core';

@Component({
  standalone: true,
  template: `
    <input 
      [(ngModel)]="name" 
      placeholder="Enter name" />
    <p>Hello, {{ name() }}!</p>
  `
})
export class ModelComponent {
  name = model('World');
}
```

## Event-Driven Updates

### DOM Events

```typescript
@Component({
  standalone: true,
  template: `
    <button (click)="handleClick($event)">Click me</button>
    <input (input)="handleInput($event)" />
  `
})
export class EventComponent {
  clicks = signal(0);

  handleClick(event: MouseEvent) {
    this.clicks.set(this.clicks() + 1);
    console.log('Button clicked:', event);
  }

  handleInput(event: Event) {
    const input = event.target as HTMLInputElement;
    console.log('Input value:', input.value);
  }
}
```

### HTTP Requests

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class DataService {
  data = signal<any[]>([]);

  constructor(private http: HttpClient) {}

  loadData() {
    // HTTP requests work without Zone.js in zoneless apps
    this.http.get<any[]>('/api/data').subscribe({
      next: (response) => {
        // Signal update triggers change detection
        this.data.set(response);
      },
      error: (error) => {
        console.error('Failed to load data:', error);
      }
    });
  }
}
```

### Timer-Based Updates

```typescript
@Component({
  standalone: true,
  template: `
    <p>Time: {{ currentTime() }}</p>
    <button (click)="startTimer()">Start</button>
    <button (click)="stopTimer()">Stop</button>
  `
})
export class TimerComponent {
  currentTime = signal(new Date());
  private timerId: any = null;

  startTimer() {
    // Use setInterval directly - no Zone.js needed
    this.timerId = setInterval(() => {
      // Signal update triggers change detection
      this.currentTime.set(new Date());
    }, 1000);
  }

  stopTimer() {
    if (this.timerId) {
      clearInterval(this.timerId);
      this.timerId = null;
    }
  }

  ngOnDestroy() {
    this.stopTimer();
  }
}
```

## Migration Strategies

### Gradual Migration

1. **Add Signals to existing components**
2. **Convert components to OnPush**
3. **Remove Zone.js gradually**

```typescript
// Step 1: Add Signals
export class HybridComponent {
  data = signal([]);
  // ... existing zone-based code
}

// Step 2: Use OnPush
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  // ... rest of component
})
export class HybridComponent {
  // ... component with signals and OnPush
}

// Step 3: Remove Zone.js (app.config.ts)
export const appConfig: ApplicationConfig = {
  providers: [
    // Remove provideZoneChangeDetection
  ]
};
```

### Complete Migration Plan

```bash
# 1. Update to Angular 21
ng update @angular/core @angular/cli

# 2. Run migration tool
ng g @schematics/angular:onpush-zoneless-migration

# 3. Review and fix issues
ng build
ng test

# 4. Remove Zone.js dependencies
npm uninstall zone.js
```

## Common Patterns

### Observable to Signal Conversion

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  standalone: true,
  template: `
    <p>Users loaded: {{ users().length }}</p>
    @for (user of users(); track user.id) {
      <p>{{ user.name }}</p>
    }
  `
})
export class UserListComponent {
  // Convert Observable to Signal
  users = toSignal(
    this.userService.getUsers(),
    { initialValue: [] }
  );

  constructor(private userService: UserService) {}
}
```

### Manual Change Detection (When Needed)

```typescript
import { ChangeDetectorRef } from '@angular/core';

@Component({
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ManualUpdateComponent {
  data = [];

  constructor(private cdr: ChangeDetectorRef) {}

  updateData() {
    // Fetch data from non-Angular source
    fetch('/api/data').then(response => {
      this.data = response;
      // Manually trigger change detection
      this.cdr.detectChanges();
    });
  }
}
```

### Third-Party Library Integration

```typescript
@Component({
  standalone: true,
  template: `
    <div #chartContainer></div>
  `
})
export class ChartComponent {
  chartData = signal([]);

  constructor(private el: ElementRef) {}

  ngAfterViewInit() {
    // Initialize third-party chart library
    const chart = new Chart(this.el.nativeElement, {
      data: this.chartData()
    });

    // Update chart when data changes
    effect(() => {
      chart.data = this.chartData();
      chart.update();
    });
  }
}
```

## Performance Benefits

### Bundle Size Reduction

```
Before (Zone.js):  ~42KB gzipped
After (Zoneless):    ~2KB gzipped
Savings:            ~40KB (95% reduction)
```

### Runtime Performance

```typescript
// Zone.js overhead
function zoneWrappedFunction() {
  // Zone.js tracking overhead here
  // Lots of bookkeeping
  // Performance impact
}

// Zoneless function
function directFunction() {
  // No overhead
  // Direct execution
  // Better performance
}
```

### Core Web Vitals Improvement

- **LCP**: Faster initial paint (no Zone.js bootstrapping)
- **FID**: Better input responsiveness (no Zone.js overhead)
- **CLS**: More stable layout (explicit updates)

## Debugging Zoneless Apps

### Clearer Stack Traces

```typescript
// Zone.js stack trace (complex)
ZoneTask.invokeTask
ZoneDelegate.invokeTask
NgZone.runTask
ApplicationRef.tick
AppComponent.myMethod

// Zoneless stack trace (clear)  
AppComponent.myMethod
ComponentRef.detectChanges
```

### Signal Debugging

```typescript
// Enable signal debugging in dev mode
import { enableSignalDevMode } from '@angular/core';

if (isDevMode()) {
  enableSignalDevMode();
}

// Now you can inspect signals in DevTools
console.log('Signal value:', mySignal());
```

## Testing Zoneless Applications

### Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import { AngularViteConfig } from '@angular/build';

export default defineConfig(AngularViteConfig({
  test: {
    environment: 'jsdom',
    // Zoneless apps work seamlessly with Vitest
  }
}));
```

### Testing Signals

```typescript
import { TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';

describe('CounterComponent (Zoneless)', () => {
  it('should update display when signal changes', () => {
    const fixture = TestBed.createComponent(CounterComponent);
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('0');

    // Signal update automatically triggers update in tests
    fixture.componentInstance.count.set(5);
    fixture.detectChanges();

    expect(fixture.nativeElement.textContent).toContain('5');
  });
});
```

## Limitations and Considerations

### When Zone.js Might Be Needed

- **Legacy third-party libraries** that expect Zone.js
- **Complex timing-dependent code**
- **Migration periods** (hybrid applications)

### Workarounds

```typescript
// For legacy code that needs Zone.js
import { NgZone } from '@angular/core';

@Component({...})
export class LegacyInteropComponent {
  constructor(private ngZone: NgZone) {}

  runLegacyCode() {
    // Run code outside Zone.js
    this.ngZone.runOutsideAngular(() => {
      // Legacy code here
    });

    // Or run inside Zone.js
    this.ngZone.run(() => {
      // Zone.js-dependent code here
    });
  }
}
```

## Best Practices

### 1. Embrace Signals Fully

```typescript
// Good: Signal-based
@Component({...})
export class GoodComponent {
  data = signal([]);
  filtered = computed(() => 
    this.data().filter(item => item.active)
  );
}

// Avoid: Mixed patterns
@Component({...})  
export class AvoidComponent {
  data = []; // Not reactive
  filteredData = []; // Manual sync needed
}
```

### 2. Use Effects Sparingly

```typescript
// Good: For side effects
effect(() => {
  this.saveToServer(this.data());
});

// Avoid: For derived data
effect(() => {
  this.filtered = this.data().filter(...); // Use computed instead
});
```

### 3. Template-Driven Updates

```typescript
// Good: Events in template
<button (click)="increment()">{{ count() }}</button>

// Avoid: Manual DOM manipulation
ngAfterViewInit() {
  this.button.addEventListener('click', () => {
    this.count.set(this.count() + 1);
    // Would need manual change detection
  });
}
```

## Migration Checklist

- [ ] **Update to Angular 21**
- [ ] **Run migration tool**: `ng g @schematics/angular:onpush-zoneless-migration`
- [ ] **Replace Zone.js patterns with Signals**
- [ ] **Update HTTP calls** (they work without changes)
- [ ] **Update timer usage** (use setInterval directly)
- [ ] **Update third-party integrations**
- [ ] **Remove zone.js dependency**
- [ ] **Test thoroughly**
- [ ] **Monitor performance improvements**

## Summary

| Benefit | Description |
|---------|-------------|
| **Smaller Bundles** | No Zone.js overhead (~40KB saved) |
| **Better Performance** | No automatic patching overhead |
| **Clearer Debugging** | Simpler stack traces |
| **Modern Standards** | Uses native browser APIs |
| **Predictable Updates** | Explicit signal-based reactivity |
| **Better Core Web Vitals** | Faster LCP, FID improvements |

### Getting Started

1. **New apps**: Zoneless by default in Angular 21
2. **Existing apps**: Use migration tool
3. **Gradual adoption**: Mix zoneless and zone-based during transition

### Community Resources

- [Angular Zoneless Guide](https://angular.dev/guide/zoneless)
- [Migration Tools](https://angular.dev/reference/migrations/zoneless)
- [Performance Case Studies](https://angular.dev/performance)

## Next Steps

- [Signals](./12-signals.md) - Deep dive into signal-based reactivity
- [Performance Optimization](./25-performance.md) - Advanced optimization techniques

---

[Previous: Signals](./12-signals.md) | [Back to Index](./README.md) | [Next: Performance Optimization](./25-performance.md) (Chapter not yet available)