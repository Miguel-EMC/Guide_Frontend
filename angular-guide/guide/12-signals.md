# Angular Signals

Signals are Angular's primitive for reactive state management. They provide fine-grained reactivity and are the foundation of modern Angular applications.

## What are Signals?

Signals are reactive primitives that:
- Hold a value that can change over time
- Notify consumers when the value changes
- Enable fine-grained change detection
- Work seamlessly with Angular's template system

## Creating Signals

### Basic Signal

```typescript
import { signal } from '@angular/core';

// Create a signal with initial value
const count = signal(0);

// Read value (call the signal)
console.log(count()); // 0

// Write value
count.set(5);
console.log(count()); // 5

// Update based on current value
count.update(current => current + 1);
console.log(count()); // 6
```

### Typed Signals

```typescript
import { signal } from '@angular/core';
// Explicit type
const user = signal<User | null>(null);

// Array
const items = signal<string[]>([]);

// Object
interface Todo {
  id: number;
  title: string;
  completed: boolean;
}
const todos = signal<Todo[]>([]);
```

## Signal Methods

| Method | Description |
|--------|-------------|
| `signal()` | Read current value |
| `set(value)` | Replace entire value |
| `update(fn)` | Update based on current value |
| `asReadonly()` | Get read-only version |

### set() vs update()

```typescript
import { signal } from '@angular/core';
const user = signal({ name: 'John', age: 30 });

// set() - Replace entire value
user.set({ name: 'Jane', age: 25 });

// update() - Transform current value
user.update(current => ({
  ...current,
  age: current.age + 1
}));
```

### Read-only Signals

```typescript
import { Injectable, signal } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class UserService {
  // Private writable signal
  private _currentUser = signal<User | null>(null);

  // Public read-only signal
  readonly currentUser = this._currentUser.asReadonly();

  login(user: User) {
    this._currentUser.set(user);
  }

  logout() {
    this._currentUser.set(null);
  }
}
```

## Computed Signals

Computed signals derive values from other signals automatically.

```typescript
import { signal, computed } from '@angular/core';

const firstName = signal('John');
const lastName = signal('Doe');

// Computed from other signals
const fullName = computed(() => `${firstName()} ${lastName()}`);

console.log(fullName()); // 'John Doe'

firstName.set('Jane');
console.log(fullName()); // 'Jane Doe' (automatically updated)
```

### Complex Computations

```typescript
import { signal, computed } from '@angular/core';
interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

const cartItems = signal<CartItem[]>([]);
const discount = signal(0);

// Computed values
const subtotal = computed(() =>
  cartItems().reduce((sum, item) => sum + item.price * item.quantity, 0)
);

const discountAmount = computed(() =>
  subtotal() * (discount() / 100)
);

const total = computed(() =>
  subtotal() - discountAmount()
);

const itemCount = computed(() =>
  cartItems().reduce((sum, item) => sum + item.quantity, 0)
);

const isEmpty = computed(() => cartItems().length === 0);
```

### Computed with Dependencies

```typescript
import { signal, computed } from '@angular/core';
const users = signal<User[]>([]);
const searchTerm = signal('');
const selectedRole = signal<string | null>(null);

const filteredUsers = computed(() => {
  let result = users();

  // Filter by search term
  const term = searchTerm().toLowerCase();
  if (term) {
    result = result.filter(u =>
      u.name.toLowerCase().includes(term) ||
      u.email.toLowerCase().includes(term)
    );
  }

  // Filter by role
  const role = selectedRole();
  if (role) {
    result = result.filter(u => u.role === role);
  }

  return result;
});
```

## Effects

Effects run side effects when signals they read change.

```typescript
import { signal, effect } from '@angular/core';

const count = signal(0);

// Effect runs when count changes
effect(() => {
  console.log('Count changed to:', count());
});

count.set(1); // Logs: "Count changed to: 1"
count.set(2); // Logs: "Count changed to: 2"
```

### Effect Use Cases

```typescript
import { Component, inject, signal, effect } from '@angular/core';
import { HttpClient } from '@angular/common/http';
@Component({...})
export class DataComponent {
  private http = inject(HttpClient);

  searchTerm = signal('');
  results = signal<SearchResult[]>([]);

  constructor() {
    // Auto-search when term changes
    effect(() => {
      const term = this.searchTerm();
      if (term.length >= 3) {
        this.search(term);
      }
    });

    // Persist to localStorage
    effect(() => {
      localStorage.setItem('searchTerm', this.searchTerm());
    });

    // Analytics
    effect(() => {
      const resultCount = this.results().length;
      analytics.track('search_results', { count: resultCount });
    });
  }

  private search(term: string) {
    // Perform search
  }
}
```

### Effect Cleanup

```typescript
import { effect } from '@angular/core';
effect((onCleanup) => {
  const subscription = someObservable$.subscribe(value => {
    // Handle value
  });

  // Cleanup when effect re-runs or component destroys
  onCleanup(() => {
    subscription.unsubscribe();
  });
});

// With intervals
effect((onCleanup) => {
  const interval = setInterval(() => {
    console.log('Tick');
  }, 1000);

  onCleanup(() => clearInterval(interval));
});
```

### Effect Options

```typescript
import { effect } from '@angular/core';
// Manual cleanup (no auto-destroy)
const effectRef = effect(() => {
  console.log(count());
}, { manualCleanup: true });

// Later: clean up manually
effectRef.destroy();

// Allow signal writes in effect (use sparingly)
effect(() => {
  if (count() > 10) {
    otherSignal.set('high');
  }
}, { allowSignalWrites: true });
```

## Signals in Components

```typescript
import { Component, signal, computed } from '@angular/core';
@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <div class="counter">
      <h2>Count: {{ count() }}</h2>
      <p>Doubled: {{ doubled() }}</p>
      <p>Status: {{ status() }}</p>

      <button (click)="decrement()">-</button>
      <button (click)="increment()">+</button>
      <button (click)="reset()">Reset</button>
    </div>
  `
})
export class CounterComponent {
  count = signal(0);

  doubled = computed(() => this.count() * 2);

  status = computed(() => {
    const c = this.count();
    if (c < 0) return 'Negative';
    if (c === 0) return 'Zero';
    if (c < 10) return 'Low';
    return 'High';
  });

  increment() {
    this.count.update(c => c + 1);
  }

  decrement() {
    this.count.update(c => c - 1);
  }

  reset() {
    this.count.set(0);
  }
}
```

## Signal Inputs

```typescript
import { Component, input, computed } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `
    <div class="card">
      <h3>{{ fullName() }}</h3>
      <p>{{ user().email }}</p>
      @if (showDetails()) {
        <p>Age: {{ user().age }}</p>
      }
    </div>
  `
})
export class UserCardComponent {
  // Required input
  user = input.required<User>();

  // Optional input with default
  showDetails = input(false);

  // Computed from input
  fullName = computed(() => {
    const u = this.user();
    return `${u.firstName} ${u.lastName}`;
  });
}
```

## Signal Outputs

```typescript
import { Component, output, signal } from '@angular/core';

@Component({
  selector: 'app-search',
  template: `
    <input
      [value]="searchTerm()"
      (input)="onInput($event)"
      (keyup.enter)="onSearch()"
    />
    <button (click)="onSearch()">Search</button>
  `
})
export class SearchComponent {
  searchTerm = signal('');

  // Signal output
  search = output<string>();

  onInput(event: Event) {
    this.searchTerm.set((event.target as HTMLInputElement).value);
  }

  onSearch() {
    this.search.emit(this.searchTerm());
  }
}
```

## Signal Model (Two-way Binding)

```typescript
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-rating',
  template: `
    @for (star of [1, 2, 3, 4, 5]; track star) {
      <button
        (click)="value.set(star)"
        [class.active]="star <= value()"
      >
        ★
      </button>
    }
  `
})
export class RatingComponent {
  value = model(0);
}

// Usage
// <app-rating [(value)]="rating" />
```

## linkedSignal

Create a signal that syncs with another signal:

```typescript
import { signal, linkedSignal } from '@angular/core';

const source = signal(5);

// Linked signal follows source
const linked = linkedSignal(() => source() * 2);

console.log(linked()); // 10

source.set(10);
console.log(linked()); // 20

// Can still set independently
linked.set(100);
console.log(linked()); // 100
```

## resource()

Load async data as signals:

```typescript
import { resource, signal, Component } from '@angular/core';

@Component({...})
export class UserProfileComponent {
  userId = signal(1);

  userResource = resource({
    request: () => ({ id: this.userId() }),
    loader: async ({ request }) => {
      const response = await fetch(`/api/users/${request.id}`);
      return response.json();
    }
  });

  // Access in template
  // userResource.value() - the data
  // userResource.status() - 'idle' | 'loading' | 'error' | 'resolved'
  // userResource.error() - error if failed
}
```

```html
@switch (userResource.status()) {
  @case ('loading') {
    <p>Loading...</p>
  }
  @case ('error') {
    <p>Error: {{ userResource.error() }}</p>
  }
  @case ('resolved') {
    <div>{{ userResource.value()?.name }}</div>
  }
}
```

## Signals with RxJS

### toSignal

Convert Observable to Signal:

```typescript
import { Component } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { map } from 'rxjs/operators';
import { interval } from 'rxjs';

@Component({...})
export class TimeComponent {
  private time$ = interval(1000).pipe(
    map(() => new Date().toLocaleTimeString())
  );

  // Convert to signal
  time = toSignal(this.time$, { initialValue: '' });

  // In template: {{ time() }}
}
```

### toObservable

Convert Signal to Observable:

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { switchMap, debounceTime, distinctUntilChanged } from 'rxjs/operators';

@Component({...})
export class SearchComponent {
  searchTerm = signal('');

  // Convert to observable for debounce
  searchTerm$ = toObservable(this.searchTerm).pipe(
    debounceTime(300),
    distinctUntilChanged()
  );

  results = toSignal(
    this.searchTerm$.pipe(
      switchMap(term => this.searchService.search(term))
    ),
    { initialValue: [] }
  );
}
```

## State Management with Signals

### Simple Store

```typescript
import { Injectable, computed, signal } from '@angular/core';
@Injectable({ providedIn: 'root' })
export class TodoStore {
  // State
  private _todos = signal<Todo[]>([]);
  private _filter = signal<'all' | 'active' | 'completed'>('all');

  // Selectors
  readonly todos = this._todos.asReadonly();
  readonly filter = this._filter.asReadonly();

  readonly filteredTodos = computed(() => {
    const todos = this._todos();
    const filter = this._filter();

    switch (filter) {
      case 'active':
        return todos.filter(t => !t.completed);
      case 'completed':
        return todos.filter(t => t.completed);
      default:
        return todos;
    }
  });

  readonly stats = computed(() => ({
    total: this._todos().length,
    active: this._todos().filter(t => !t.completed).length,
    completed: this._todos().filter(t => t.completed).length
  }));

  // Actions
  addTodo(title: string) {
    const todo: Todo = {
      id: Date.now(),
      title,
      completed: false
    };
    this._todos.update(todos => [...todos, todo]);
  }

  toggleTodo(id: number) {
    this._todos.update(todos =>
      todos.map(t =>
        t.id === id ? { ...t, completed: !t.completed } : t
      )
    );
  }

  removeTodo(id: number) {
    this._todos.update(todos => todos.filter(t => t.id !== id));
  }

  setFilter(filter: 'all' | 'active' | 'completed') {
    this._filter.set(filter);
  }

  clearCompleted() {
    this._todos.update(todos => todos.filter(t => !t.completed));
  }
}
```

### Using the Store

```typescript
import { Component, inject } from '@angular/core';
@Component({
  template: `
    <div class="filters">
      <button
        (click)="store.setFilter('all')"
        [class.active]="store.filter() === 'all'"
      >All ({{ store.stats().total }})</button>
      <button
        (click)="store.setFilter('active')"
        [class.active]="store.filter() === 'active'"
      >Active ({{ store.stats().active }})</button>
      <button
        (click)="store.setFilter('completed')"
        [class.active]="store.filter() === 'completed'"
      >Completed ({{ store.stats().completed }})</button>
    </div>

    @for (todo of store.filteredTodos(); track todo.id) {
      <div class="todo" [class.completed]="todo.completed">
        <input
          type="checkbox"
          [checked]="todo.completed"
          (change)="store.toggleTodo(todo.id)"
        />
        <span>{{ todo.title }}</span>
        <button (click)="store.removeTodo(todo.id)">×</button>
      </div>
    }
  `
})
export class TodoListComponent {
  store = inject(TodoStore);
}
```

## Best Practices

### Do's

```typescript
import { signal, computed, effect } from '@angular/core';
import { last, first } from 'rxjs/operators';
// ✅ Use computed for derived values
const fullName = computed(() => `${first()} ${last()}`);

// ✅ Use update for transformations
count.update(c => c + 1);

// ✅ Keep signals private when possible
private _data = signal([]);
readonly data = this._data.asReadonly();

// ✅ Use effects for side effects only
effect(() => {
  localStorage.setItem('data', JSON.stringify(data()));
});
```

### Don'ts

```typescript
import { signal, computed } from '@angular/core';
// ❌ Don't call signals conditionally in computed
const bad = computed(() => {
  if (condition) {
    return value(); // May not track properly
  }
  return 0;
});

// ❌ Don't modify signals in computed
const bad = computed(() => {
  count.set(1); // Error!
  return count();
});

// ❌ Don't create signals inside computed
const bad = computed(() => {
  const temp = signal(0); // Bad!
  return temp();
});
```

## Summary

| Concept | Description |
|---------|-------------|
| `signal()` | Create reactive value |
| `computed()` | Derived reactive value |
| `effect()` | Run side effects |
| `input()` | Signal-based input |
| `output()` | Signal-based output |
| `model()` | Two-way binding signal |
| `toSignal()` | Observable to Signal |
| `toObservable()` | Signal to Observable |

## Next Steps

- [RxJS & Observables](./13-rxjs.md) - Async patterns with RxJS
- [State Management](./14-state-management.md) - State management patterns

---

[Previous: HTTP Client](./11-http-client.md) | [Back to Index](./README.md) | [Next: RxJS & Observables](./13-rxjs.md)
