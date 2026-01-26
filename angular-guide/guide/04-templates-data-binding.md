# Templates & Data Binding

Templates define the view of Angular components. They use HTML enriched with Angular's template syntax for dynamic content.

## Interpolation

Display component data in the template using double curly braces.

```html
<!-- Simple property -->
<h1>{{ title }}</h1>

<!-- Signal (call to get value) -->
<p>{{ count() }}</p>

<!-- Expressions -->
<p>{{ firstName + ' ' + lastName }}</p>
<p>{{ price * quantity }}</p>
<p>{{ isActive ? 'Active' : 'Inactive' }}</p>

<!-- Method calls -->
<p>{{ getFullName() }}</p>
<p>{{ items.length }}</p>
<p>{{ message.toUpperCase() }}</p>

<!-- Nested properties -->
<p>{{ user.address.city }}</p>
<p>{{ user?.address?.city }}</p> <!-- Safe navigation -->
```

### Limitations

```html
<!-- NOT allowed in interpolation -->
{{ count++ }}           <!-- No assignments -->
{{ new Date() }}        <!-- No new -->
{{ console.log() }}     <!-- No global objects -->
{{ a; b }}              <!-- No chained expressions -->
```

## Property Binding

Bind component properties to element properties or attributes.

### Element Properties

```html
<!-- Bind to property -->
<img [src]="imageUrl" [alt]="imageAlt" />
<input [value]="username" />
<button [disabled]="isLoading()">Submit</button>

<!-- Bind to innerHTML -->
<div [innerHTML]="htmlContent"></div>

<!-- Equivalent syntax -->
<img bind-src="imageUrl" />
```

### Attribute Binding

Use `attr.` prefix for attributes that don't have corresponding properties.

```html
<!-- ARIA attributes -->
<button [attr.aria-label]="buttonLabel">
  <span [attr.aria-hidden]="true">Icon</span>
</button>

<!-- Data attributes -->
<div [attr.data-id]="item.id"></div>

<!-- Colspan (attribute, not property) -->
<td [attr.colspan]="columnSpan"></td>

<!-- Remove attribute when null -->
<input [attr.disabled]="isDisabled ? '' : null" />
```

### Class Binding

```html
<!-- Single class toggle -->
<div [class.active]="isActive()"></div>
<div [class.error]="hasError"></div>
<div [class.is-loading]="isLoading()"></div>

<!-- Multiple classes (object) -->
<div [class]="{ active: isActive(), error: hasError }"></div>

<!-- Class from property -->
<div [class]="currentClasses"></div>

<!-- NgClass directive (requires import) -->
<div [ngClass]="{
  'active': isActive(),
  'disabled': isDisabled(),
  'highlighted': isHighlighted()
}"></div>
```

### Style Binding

```html
<!-- Single style -->
<div [style.color]="textColor"></div>
<div [style.width.px]="width"></div>
<div [style.height.%]="heightPercent"></div>
<div [style.font-size.em]="fontSize"></div>

<!-- Multiple styles (object) -->
<div [style]="{
  'color': textColor,
  'font-size.px': fontSize
}"></div>

<!-- NgStyle directive -->
<div [ngStyle]="{
  'background-color': bgColor,
  'border': border
}"></div>
```

## Event Binding

Listen to DOM events and component outputs.

### DOM Events

```html
<!-- Click -->
<button (click)="onClick()">Click me</button>
<button (click)="onClick($event)">With event</button>

<!-- Input events -->
<input (input)="onInput($event)" />
<input (change)="onChange($event)" />
<input (focus)="onFocus()" />
<input (blur)="onBlur()" />

<!-- Keyboard events -->
<input (keyup)="onKeyUp($event)" />
<input (keydown)="onKeyDown($event)" />
<input (keyup.enter)="onEnter()" />
<input (keyup.escape)="onEscape()" />
<input (keydown.control.s)="onSave($event)" />

<!-- Mouse events -->
<div (mouseenter)="onMouseEnter()"></div>
<div (mouseleave)="onMouseLeave()"></div>
<div (mousemove)="onMouseMove($event)"></div>

<!-- Form events -->
<form (ngSubmit)="onSubmit()"></form>
<select (change)="onSelect($event)"></select>
```

### Event Object

```typescript
// Component
onInput(event: Event) {
  const input = event.target as HTMLInputElement;
  console.log(input.value);
}

onClick(event: MouseEvent) {
  console.log('Click position:', event.clientX, event.clientY);
}

onKeyDown(event: KeyboardEvent) {
  console.log('Key pressed:', event.key);
  if (event.key === 'Enter') {
    event.preventDefault();
  }
}
```

### Key Modifiers

| Modifier | Description |
|----------|-------------|
| `enter` | Enter key |
| `escape` | Escape key |
| `space` | Spacebar |
| `tab` | Tab key |
| `arrowup` | Arrow up |
| `arrowdown` | Arrow down |
| `control` | Ctrl key |
| `alt` | Alt key |
| `shift` | Shift key |
| `meta` | Cmd (Mac) / Win (Windows) |

```html
<!-- Combinations -->
<input (keydown.control.enter)="submitForm()" />
<input (keydown.shift.tab)="previousField()" />
<input (keydown.alt.backspace)="clearField()" />
```

## Two-Way Binding

Combine property and event binding for two-way data flow.

### NgModel (FormsModule)

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-form',
  standalone: true,
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

### Custom Two-Way Binding

The banana-in-a-box syntax `[()]` is shorthand for property + event:

```html
<!-- This: -->
<input [(ngModel)]="username" />

<!-- Is equivalent to: -->
<input [ngModel]="username" (ngModelChange)="username = $event" />
```

### With model() (v21)

```typescript
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-slider',
  standalone: true,
  template: `
    <input
      type="range"
      [value]="value()"
      (input)="value.set($any($event.target).valueAsNumber)"
    />
    <span>{{ value() }}</span>
  `
})
export class SliderComponent {
  value = model(50);
}

// Parent usage:
// <app-slider [(value)]="myValue" />
```

## Template Reference Variables

Create references to DOM elements or components.

### Element References

```html
<!-- Create reference -->
<input #nameInput type="text" />
<button (click)="greet(nameInput.value)">Greet</button>

<!-- Access in template -->
<input #emailInput type="email" />
<p>Email length: {{ emailInput.value.length }}</p>

<!-- Multiple references -->
<input #firstName />
<input #lastName />
<button (click)="submit(firstName.value, lastName.value)">Submit</button>
```

### Component References

```html
<!-- Reference a component -->
<app-timer #timer></app-timer>
<button (click)="timer.start()">Start</button>
<button (click)="timer.stop()">Stop</button>
<p>Elapsed: {{ timer.elapsed() }}</p>
```

### Access in Component

```typescript
import { Component, viewChild, ElementRef } from '@angular/core';

@Component({
  template: `<input #searchInput />`
})
export class SearchComponent {
  searchInput = viewChild<ElementRef>('searchInput');

  focusInput() {
    this.searchInput()?.nativeElement.focus();
  }
}
```

## Template Expressions

### Operators

```html
<!-- Arithmetic -->
<p>{{ price * quantity }}</p>
<p>{{ total + tax }}</p>

<!-- Comparison -->
<p>{{ count > 10 ? 'Many' : 'Few' }}</p>

<!-- Logical -->
<p>{{ isLoggedIn && hasPermission }}</p>
<p>{{ name || 'Anonymous' }}</p>

<!-- Nullish coalescing -->
<p>{{ user?.name ?? 'Unknown' }}</p>

<!-- Optional chaining -->
<p>{{ user?.address?.city }}</p>
```

### Pipes in Expressions

```html
<!-- Format data -->
<p>{{ birthday | date:'fullDate' }}</p>
<p>{{ price | currency:'USD' }}</p>
<p>{{ name | uppercase }}</p>

<!-- Chained pipes -->
<p>{{ birthday | date:'short' | uppercase }}</p>

<!-- With async -->
<p>{{ data$ | async }}</p>
<p>{{ (users$ | async)?.length }}</p>
```

## Built-in Control Flow

### @if

```html
@if (condition) {
  <p>Condition is true</p>
}

@if (user()) {
  <p>Welcome, {{ user()!.name }}</p>
} @else if (isLoading()) {
  <app-spinner />
} @else {
  <p>Please log in</p>
}

<!-- With reference -->
@if (user(); as currentUser) {
  <p>Welcome, {{ currentUser.name }}</p>
}
```

### @for

```html
@for (item of items(); track item.id) {
  <div>{{ item.name }}</div>
}

<!-- With index and other variables -->
@for (item of items(); track item.id; let i = $index) {
  <div>{{ i + 1 }}. {{ item.name }}</div>
}

<!-- Available variables -->
@for (item of items(); track item.id; let i = $index, first = $first, last = $last, even = $even, odd = $odd, count = $count) {
  <div
    [class.first]="first"
    [class.last]="last"
    [class.even]="even"
    [class.odd]="odd"
  >
    {{ i + 1 }} of {{ count }}: {{ item.name }}
  </div>
}

<!-- Empty state -->
@for (item of items(); track item.id) {
  <div>{{ item.name }}</div>
} @empty {
  <p>No items found</p>
}
```

### @switch

```html
@switch (status()) {
  @case ('pending') {
    <span class="badge pending">Pending</span>
  }
  @case ('approved') {
    <span class="badge approved">Approved</span>
  }
  @case ('rejected') {
    <span class="badge rejected">Rejected</span>
  }
  @default {
    <span class="badge">Unknown</span>
  }
}
```

### @defer

Lazy load parts of the template:

```html
@defer (on viewport) {
  <app-heavy-chart [data]="chartData()" />
} @placeholder {
  <div class="chart-placeholder">Chart will load...</div>
} @loading (minimum 200ms) {
  <app-spinner />
} @error {
  <p class="error">Failed to load chart</p>
}
```

**Triggers:**

```html
<!-- Load when browser is idle -->
@defer (on idle) { ... }

<!-- Load when element enters viewport -->
@defer (on viewport) { ... }
@defer (on viewport(rootMargin: 100px)) { ... }

<!-- Load on interaction -->
@defer (on interaction) { ... }
@defer (on interaction(button)) { ... }

<!-- Load on hover -->
@defer (on hover) { ... }
@defer (on hover(trigger)) { ... }

<!-- Load immediately (still separate chunk) -->
@defer (on immediate) { ... }

<!-- Load after delay -->
@defer (on timer(5s)) { ... }

<!-- Load when condition is true -->
@defer (when shouldLoad()) { ... }

<!-- Prefetch -->
@defer (on interaction; prefetch on idle) { ... }
```

## @let for Template Variables

Declare local variables in templates (v21):

```html
<!-- Simple assignment -->
@let greeting = 'Hello, ' + name();
<h1>{{ greeting }}</h1>

<!-- With computed values -->
@let total = items().reduce((sum, item) => sum + item.price, 0);
<p>Total: {{ total | currency }}</p>

<!-- With async -->
@let user = user$ | async;
@if (user) {
  <p>Welcome, {{ user.name }}</p>
}

<!-- Complex expressions -->
@let fullAddress = user()?.address?.street + ', ' + user()?.address?.city;
<p>{{ fullAddress }}</p>
```

## NgTemplateOutlet

Render templates dynamically:

```typescript
@Component({
  selector: 'app-list',
  standalone: true,
  imports: [NgTemplateOutlet],
  template: `
    <ul>
      @for (item of items(); track item.id) {
        <li>
          <ng-container
            [ngTemplateOutlet]="itemTemplate() || defaultTemplate"
            [ngTemplateOutletContext]="{ $implicit: item, index: $index }"
          />
        </li>
      }
    </ul>

    <ng-template #defaultTemplate let-item>
      {{ item.name }}
    </ng-template>
  `
})
export class ListComponent {
  items = input.required<any[]>();
  itemTemplate = input<TemplateRef<any>>();
}
```

**Usage:**

```html
<app-list [items]="products()">
  <ng-template let-product let-i="index">
    <strong>{{ i + 1 }}.</strong> {{ product.name }} - {{ product.price | currency }}
  </ng-template>
</app-list>
```

## Ng-Container

Grouping element that doesn't render:

```html
<!-- Group elements without wrapper -->
<ng-container>
  <dt>Name</dt>
  <dd>{{ user.name }}</dd>
</ng-container>

<!-- With control flow -->
@for (item of items(); track item.id) {
  <ng-container>
    @if (item.visible) {
      <div>{{ item.name }}</div>
    }
  </ng-container>
}

<!-- With structural directives -->
<ng-container *ngIf="user as u">
  <p>{{ u.name }}</p>
  <p>{{ u.email }}</p>
</ng-container>
```

## Summary

| Syntax | Purpose | Example |
|--------|---------|---------|
| `{{ }}` | Interpolation | `{{ title }}` |
| `[property]` | Property binding | `[disabled]="isDisabled"` |
| `(event)` | Event binding | `(click)="onClick()"` |
| `[(ngModel)]` | Two-way binding | `[(ngModel)]="name"` |
| `#ref` | Template reference | `#input` |
| `@if` | Conditional | `@if (show) { ... }` |
| `@for` | Loop | `@for (item of items; track item.id)` |
| `@switch` | Switch | `@switch (value) { @case ... }` |
| `@defer` | Lazy loading | `@defer (on viewport) { ... }` |
| `@let` | Local variable | `@let total = sum()` |

## Next Steps

- [Directives](./05-directives.md) - Built-in and custom directives
- [Pipes](./06-pipes.md) - Transform displayed data

---

[Previous: Components](./03-components.md) | [Back to Index](./README.md) | [Next: Directives](./05-directives.md)
