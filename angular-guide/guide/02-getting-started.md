# Getting Started

This section covers creating your first components, understanding the development workflow, and building interactive features.

## Creating Your First Component

### Generate a Component

```bash
ng generate component hello-world
# or shorthand
ng g c hello-world
```

**Generated files:**

```
src/app/hello-world/
├── hello-world.component.ts       # Component logic
├── hello-world.component.html     # Template
├── hello-world.component.scss     # Styles
└── hello-world.component.spec.ts  # Tests
```

### Understanding the Component

**hello-world.component.ts:**

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-hello-world',
  standalone: true,
  imports: [],
  templateUrl: './hello-world.component.html',
  styleUrl: './hello-world.component.scss'
})
export class HelloWorldComponent {

}
```

### Add Interactivity

**hello-world.component.ts:**

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-hello-world',
  standalone: true,
  imports: [],
  templateUrl: './hello-world.component.html',
  styleUrl: './hello-world.component.scss'
})
export class HelloWorldComponent {
  // Properties
  title = 'Hello Angular v21!';

  // Signals (reactive state)
  count = signal(0);
  name = signal('World');

  // Methods
  increment() {
    this.count.update(c => c + 1);
  }

  decrement() {
    this.count.update(c => c - 1);
  }

  updateName(newName: string) {
    this.name.set(newName);
  }
}
```

**hello-world.component.html:**

```html
<div class="container">
  <h1>{{ title }}</h1>
  <h2>Hello, {{ name() }}!</h2>

  <div class="counter">
    <p>Count: {{ count() }}</p>
    <button (click)="decrement()">-</button>
    <button (click)="increment()">+</button>
  </div>

  <div class="name-input">
    <label for="name">Your name:</label>
    <input
      id="name"
      type="text"
      [value]="name()"
      (input)="updateName($any($event.target).value)"
    />
  </div>
</div>
```

**hello-world.component.scss:**

```scss
.container {
  max-width: 600px;
  margin: 2rem auto;
  padding: 2rem;
  text-align: center;
  font-family: system-ui, sans-serif;

  h1 {
    color: #1976d2;
    margin-bottom: 1rem;
  }

  h2 {
    color: #333;
    margin-bottom: 2rem;
  }
}

.counter {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 2rem;

  p {
    font-size: 1.5rem;
    min-width: 100px;
  }

  button {
    padding: 0.5rem 1.5rem;
    font-size: 1.5rem;
    cursor: pointer;
    border: none;
    border-radius: 4px;
    background: #1976d2;
    color: white;
    transition: background 0.2s;

    &:hover {
      background: #1565c0;
    }
  }
}

.name-input {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;

  label {
    font-weight: 500;
  }

  input {
    padding: 0.75rem;
    font-size: 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    text-align: center;

    &:focus {
      outline: none;
      border-color: #1976d2;
    }
  }
}
```

### Use the Component

**app.component.ts:**

```typescript
import { Component } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { HelloWorldComponent } from './hello-world/hello-world.component';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, HelloWorldComponent],
  templateUrl: './app.component.html',
  styleUrl: './app.component.scss'
})
export class AppComponent {
  title = 'my-first-app';
}
```

**app.component.html:**

```html
<main>
  <app-hello-world />
</main>
<router-outlet />
```

## Template Syntax Basics

### Interpolation

Display component data in the template:

```html
<!-- Simple interpolation -->
<p>{{ title }}</p>

<!-- Signal interpolation (call the signal) -->
<p>{{ count() }}</p>

<!-- Expressions -->
<p>{{ 1 + 1 }}</p>
<p>{{ name().toUpperCase() }}</p>
<p>{{ items.length }}</p>
```

### Property Binding

Bind properties to DOM elements:

```html
<!-- Bind to element property -->
<img [src]="imageUrl" [alt]="imageDescription" />

<!-- Bind to input value -->
<input [value]="name()" />

<!-- Bind to disabled state -->
<button [disabled]="isLoading()">Submit</button>

<!-- Bind to class -->
<div [class.active]="isActive()">Content</div>

<!-- Bind to style -->
<div [style.color]="textColor">Styled text</div>
```

### Event Binding

Listen to DOM events:

```html
<!-- Click event -->
<button (click)="handleClick()">Click me</button>

<!-- With event parameter -->
<button (click)="handleClick($event)">Click</button>

<!-- Input event -->
<input (input)="onInput($event)" />

<!-- Keyboard event -->
<input (keyup.enter)="onEnter()" />

<!-- Focus events -->
<input (focus)="onFocus()" (blur)="onBlur()" />
```

### Two-Way Binding

Combine property and event binding:

```typescript
import { FormsModule } from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <input [(ngModel)]="username" />
    <p>Hello, {{ username }}</p>
  `
})
export class FormComponent {
  username = '';
}
```

## Control Flow (Built-in)

Angular v21 uses built-in control flow syntax (no imports needed).

### @if / @else

```html
@if (isLoggedIn()) {
  <p>Welcome back, {{ username() }}!</p>
} @else if (isLoading()) {
  <p>Loading...</p>
} @else {
  <p>Please log in</p>
}
```

### @for

```html
@for (item of items(); track item.id) {
  <div class="item">
    <h3>{{ item.name }}</h3>
    <p>{{ item.description }}</p>
  </div>
} @empty {
  <p>No items found</p>
}
```

**Important:** The `track` expression is required and improves performance.

### @switch

```html
@switch (status()) {
  @case ('loading') {
    <app-spinner />
  }
  @case ('success') {
    <app-content [data]="data()" />
  }
  @case ('error') {
    <app-error [message]="errorMessage()" />
  }
  @default {
    <p>Unknown status</p>
  }
}
```

### @defer (Lazy Loading)

```html
@defer (on viewport) {
  <app-heavy-component />
} @placeholder {
  <p>Component will load when visible...</p>
} @loading (minimum 500ms) {
  <app-spinner />
} @error {
  <p>Failed to load component</p>
}
```

**Defer triggers:**

| Trigger | Description |
|---------|-------------|
| `on idle` | When browser is idle |
| `on viewport` | When element enters viewport |
| `on interaction` | On user interaction |
| `on hover` | On mouse hover |
| `on immediate` | Immediately after render |
| `on timer(5s)` | After specified delay |
| `when condition` | When expression is true |

## Building a Todo List

Let's build a simple todo list to practice:

### Generate Component

```bash
ng g c todo-list
```

### Todo Interface

**src/app/todo-list/todo.interface.ts:**

```typescript
export interface Todo {
  id: number;
  title: string;
  completed: boolean;
  createdAt: Date;
}
```

### Todo List Component

**todo-list.component.ts:**

```typescript
import { Component, signal, computed } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { Todo } from './todo.interface';

@Component({
  selector: 'app-todo-list',
  standalone: true,
  imports: [FormsModule],
  templateUrl: './todo-list.component.html',
  styleUrl: './todo-list.component.scss'
})
export class TodoListComponent {
  // State
  todos = signal<Todo[]>([
    { id: 1, title: 'Learn Angular', completed: true, createdAt: new Date() },
    { id: 2, title: 'Build a project', completed: false, createdAt: new Date() },
    { id: 3, title: 'Deploy to production', completed: false, createdAt: new Date() }
  ]);

  newTodoTitle = '';
  filter = signal<'all' | 'active' | 'completed'>('all');

  // Computed values
  filteredTodos = computed(() => {
    const currentFilter = this.filter();
    const allTodos = this.todos();

    switch (currentFilter) {
      case 'active':
        return allTodos.filter(t => !t.completed);
      case 'completed':
        return allTodos.filter(t => t.completed);
      default:
        return allTodos;
    }
  });

  remainingCount = computed(() =>
    this.todos().filter(t => !t.completed).length
  );

  // Actions
  addTodo() {
    const title = this.newTodoTitle.trim();
    if (!title) return;

    const newTodo: Todo = {
      id: Date.now(),
      title,
      completed: false,
      createdAt: new Date()
    };

    this.todos.update(todos => [...todos, newTodo]);
    this.newTodoTitle = '';
  }

  toggleTodo(id: number) {
    this.todos.update(todos =>
      todos.map(todo =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    );
  }

  removeTodo(id: number) {
    this.todos.update(todos => todos.filter(t => t.id !== id));
  }

  clearCompleted() {
    this.todos.update(todos => todos.filter(t => !t.completed));
  }

  setFilter(filter: 'all' | 'active' | 'completed') {
    this.filter.set(filter);
  }
}
```

**todo-list.component.html:**

```html
<div class="todo-app">
  <h1>Todo List</h1>

  <!-- Add todo form -->
  <form class="add-form" (ngSubmit)="addTodo()">
    <input
      type="text"
      [(ngModel)]="newTodoTitle"
      name="newTodo"
      placeholder="What needs to be done?"
      autofocus
    />
    <button type="submit" [disabled]="!newTodoTitle.trim()">Add</button>
  </form>

  <!-- Todo list -->
  <ul class="todo-list">
    @for (todo of filteredTodos(); track todo.id) {
      <li [class.completed]="todo.completed">
        <input
          type="checkbox"
          [checked]="todo.completed"
          (change)="toggleTodo(todo.id)"
        />
        <span class="title">{{ todo.title }}</span>
        <button class="delete" (click)="removeTodo(todo.id)">×</button>
      </li>
    } @empty {
      <li class="empty">No todos to display</li>
    }
  </ul>

  <!-- Footer -->
  @if (todos().length > 0) {
    <footer class="footer">
      <span class="count">{{ remainingCount() }} items left</span>

      <div class="filters">
        <button
          [class.active]="filter() === 'all'"
          (click)="setFilter('all')"
        >All</button>
        <button
          [class.active]="filter() === 'active'"
          (click)="setFilter('active')"
        >Active</button>
        <button
          [class.active]="filter() === 'completed'"
          (click)="setFilter('completed')"
        >Completed</button>
      </div>

      @if (todos().length - remainingCount() > 0) {
        <button class="clear" (click)="clearCompleted()">
          Clear completed
        </button>
      }
    </footer>
  }
</div>
```

**todo-list.component.scss:**

```scss
.todo-app {
  max-width: 500px;
  margin: 2rem auto;
  padding: 0 1rem;
  font-family: system-ui, sans-serif;

  h1 {
    text-align: center;
    color: #1976d2;
    margin-bottom: 1.5rem;
  }
}

.add-form {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;

  input {
    flex: 1;
    padding: 0.75rem;
    font-size: 1rem;
    border: 2px solid #ddd;
    border-radius: 4px;

    &:focus {
      outline: none;
      border-color: #1976d2;
    }
  }

  button {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    background: #1976d2;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;

    &:disabled {
      background: #ccc;
      cursor: not-allowed;
    }

    &:hover:not(:disabled) {
      background: #1565c0;
    }
  }
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
  border: 1px solid #ddd;
  border-radius: 4px;

  li {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 1rem;
    border-bottom: 1px solid #eee;

    &:last-child {
      border-bottom: none;
    }

    &.completed .title {
      text-decoration: line-through;
      color: #999;
    }

    &.empty {
      justify-content: center;
      color: #999;
      font-style: italic;
    }

    input[type="checkbox"] {
      width: 1.25rem;
      height: 1.25rem;
      cursor: pointer;
    }

    .title {
      flex: 1;
    }

    .delete {
      background: none;
      border: none;
      font-size: 1.5rem;
      color: #ff4444;
      cursor: pointer;
      opacity: 0;
      transition: opacity 0.2s;

      &:hover {
        color: #cc0000;
      }
    }

    &:hover .delete {
      opacity: 1;
    }
  }
}

.footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  background: #f9f9f9;
  border: 1px solid #ddd;
  border-top: none;
  border-radius: 0 0 4px 4px;
  font-size: 0.875rem;

  .count {
    color: #666;
  }

  .filters {
    display: flex;
    gap: 0.25rem;

    button {
      padding: 0.25rem 0.5rem;
      background: none;
      border: 1px solid transparent;
      border-radius: 4px;
      cursor: pointer;

      &.active {
        border-color: #1976d2;
      }

      &:hover {
        border-color: #ddd;
      }
    }
  }

  .clear {
    background: none;
    border: none;
    color: #666;
    cursor: pointer;

    &:hover {
      color: #333;
      text-decoration: underline;
    }
  }
}
```

## Development Workflow

### Hot Module Replacement

```bash
ng serve --hmr
```

Changes update without full page reload.

### Build for Production

```bash
ng build
```

Output in `dist/` folder.

**Build options:**

| Flag | Description |
|------|-------------|
| `--configuration=production` | Production build (default) |
| `--source-map` | Generate source maps |
| `--stats-json` | Generate bundle stats |
| `--base-href=/app/` | Set base href |

### Analyze Bundle Size

```bash
ng build --stats-json
npx webpack-bundle-analyzer dist/my-app/stats.json
```

## Debugging

### Browser DevTools

1. Open DevTools (F12)
2. Install Angular DevTools extension
3. View component tree and state

### VS Code Debugging

**.vscode/launch.json:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://localhost:4200",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```

### Console Debugging

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({ /* ... */ })
export class DebugComponent {
  count = signal(0);

  constructor() {
    // Log signal changes
    effect(() => {
      console.log('Count changed:', this.count());
    });
  }
}
```

## Summary

| Concept | Description |
|---------|-------------|
| Components | Encapsulated UI building blocks |
| Signals | Reactive state management |
| Interpolation | `{{ expression }}` in templates |
| Property Binding | `[property]="value"` |
| Event Binding | `(event)="handler()"` |
| Control Flow | `@if`, `@for`, `@switch`, `@defer` |

## Next Steps

- [Components](./03-components.md) - Deep dive into components
- [Templates & Data Binding](./04-templates-data-binding.md) - Advanced templates

---

[Previous: Introduction](./01-introduction.md) | [Back to Index](./README.md) | [Next: Components](./03-components.md)
