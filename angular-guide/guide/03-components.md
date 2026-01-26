# Components

Components are the fundamental building blocks of Angular applications. Each component encapsulates its own template, styles, and logic.

## Component Anatomy

Every Angular component consists of:

| Part | Description |
|------|-------------|
| **Class** | TypeScript class with logic and data |
| **Template** | HTML that defines the view |
| **Styles** | CSS/SCSS scoped to the component |
| **Metadata** | Configuration via `@Component` decorator |

## Creating Components

### Using CLI

```bash
# Full command
ng generate component user-profile

# Shorthand
ng g c user-profile

# With options
ng g c user-profile --skip-tests --inline-style
```

**CLI Options:**

| Option | Description |
|--------|-------------|
| `--skip-tests` | Don't generate spec file |
| `--inline-template` | Template in TS file |
| `--inline-style` | Styles in TS file |
| `--flat` | No subdirectory |
| `--standalone` | Standalone component (default in v21) |
| `--export` | Export from module |

### Component Structure

```typescript
import { Component, signal, computed, input, output } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [],
  templateUrl: './user-profile.component.html',
  styleUrl: './user-profile.component.scss'
})
export class UserProfileComponent {
  // Component logic here
}
```

### Inline Template and Styles

```typescript
@Component({
  selector: 'app-inline-example',
  standalone: true,
  template: `
    <div class="container">
      <h2>{{ title }}</h2>
      <p>{{ description }}</p>
    </div>
  `,
  styles: `
    .container {
      padding: 1rem;
      border: 1px solid #ccc;
    }
    h2 {
      color: #333;
    }
  `
})
export class InlineExampleComponent {
  title = 'Inline Component';
  description = 'This uses inline template and styles';
}
```

## Component Selectors

The selector determines how the component is used in templates.

### Element Selector (Most Common)

```typescript
@Component({
  selector: 'app-user-card',
  // ...
})
```

**Usage:**
```html
<app-user-card></app-user-card>
<!-- or self-closing (Angular v21) -->
<app-user-card />
```

### Attribute Selector

```typescript
@Component({
  selector: '[appHighlight]',
  // ...
})
```

**Usage:**
```html
<div appHighlight>Highlighted content</div>
```

### Class Selector

```typescript
@Component({
  selector: '.app-widget',
  // ...
})
```

**Usage:**
```html
<div class="app-widget">Widget content</div>
```

## Component Inputs

Inputs allow parent components to pass data to child components.

### Signal Inputs (v21 Recommended)

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `
    <div class="card">
      <h3>{{ name() }}</h3>
      <p>Age: {{ age() }}</p>
      <p>Role: {{ role() }}</p>
    </div>
  `
})
export class UserCardComponent {
  // Required input
  name = input.required<string>();

  // Optional input with default
  age = input<number>(0);

  // Optional input (undefined if not provided)
  role = input<string>();
}
```

**Usage:**

```html
<app-user-card
  [name]="'John Doe'"
  [age]="25"
  [role]="'Admin'"
/>
```

### Input Options

```typescript
import { numberAttribute, booleanAttribute } from '@angular/core';

// With alias (use different name in template)
name = input.required<string>({ alias: 'userName' });

// With transform (convert string to number/boolean)
count = input(0, { transform: numberAttribute });
disabled = input(false, { transform: booleanAttribute });
```

**Usage with alias and transform:**

```html
<app-user-card userName="John" count="5" disabled />
```

### Classic @Input Decorator

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `<h3>{{ name }}</h3>`
})
export class UserCardComponent {
  @Input() name: string = '';
  @Input({ required: true }) id!: number;
  @Input({ alias: 'userName' }) name: string = '';
  @Input({ transform: booleanAttribute }) disabled: boolean = false;
}
```

## Component Outputs

Outputs allow child components to emit events to parent components.

### Signal Outputs (v21 Recommended)

```typescript
import { Component, output } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <button (click)="increment()">+</button>
    <button (click)="decrement()">-</button>
  `
})
export class CounterComponent {
  // Define outputs
  incremented = output<number>();
  decremented = output<number>();

  private count = 0;

  increment() {
    this.count++;
    this.incremented.emit(this.count);
  }

  decrement() {
    this.count--;
    this.decremented.emit(this.count);
  }
}
```

**Usage:**

```html
<app-counter
  (incremented)="onIncrement($event)"
  (decremented)="onDecrement($event)"
/>
```

```typescript
onIncrement(value: number) {
  console.log('Incremented to:', value);
}

onDecrement(value: number) {
  console.log('Decremented to:', value);
}
```

### Output with Alias

```typescript
valueChanged = output<number>({ alias: 'change' });
```

```html
<app-counter (change)="onValueChange($event)" />
```

### outputFromObservable

```typescript
import { output, outputFromObservable } from '@angular/core';
import { interval } from 'rxjs';

// Create output from Observable
tick = outputFromObservable(interval(1000));
```

### Classic @Output Decorator

```typescript
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `<button (click)="onClick()">Click</button>`
})
export class CounterComponent {
  @Output() clicked = new EventEmitter<void>();
  @Output('change') valueChanged = new EventEmitter<number>();

  onClick() {
    this.clicked.emit();
  }
}
```

## Two-Way Binding with Model

Angular v21 introduces `model()` for simplified two-way binding.

### Using model()

```typescript
import { Component, model } from '@angular/core';

@Component({
  selector: 'app-toggle',
  standalone: true,
  template: `
    <button (click)="toggle()">
      {{ checked() ? 'ON' : 'OFF' }}
    </button>
  `
})
export class ToggleComponent {
  // Two-way bindable signal
  checked = model(false);

  toggle() {
    this.checked.update(v => !v);
  }
}
```

**Usage with two-way binding:**

```html
<app-toggle [(checked)]="isEnabled" />
```

```typescript
isEnabled = signal(false);
```

### Required Model

```typescript
value = model.required<string>();
```

### Model with Alias

```typescript
value = model(0, { alias: 'count' });
```

```html
<app-counter [(count)]="myValue" />
```

## Component Lifecycle

Angular components have lifecycle hooks that let you tap into key moments.

### Lifecycle Hooks

| Hook | Description | Common Use |
|------|-------------|------------|
| `constructor` | Class instantiation | DI only |
| `ngOnChanges` | Before `ngOnInit` and when inputs change | React to input changes |
| `ngOnInit` | After first input binding | Fetch initial data |
| `ngDoCheck` | Every change detection | Custom change detection |
| `ngAfterContentInit` | After content projection | Access content children |
| `ngAfterContentChecked` | After every content check | Content change reactions |
| `ngAfterViewInit` | After view initializes | Access view children |
| `ngAfterViewChecked` | After every view check | View change reactions |
| `ngOnDestroy` | Before component destroys | Cleanup subscriptions |

### Implementation

```typescript
import {
  Component,
  OnInit,
  OnDestroy,
  OnChanges,
  AfterViewInit,
  SimpleChanges,
  input,
  signal
} from '@angular/core';
import { Subscription, interval } from 'rxjs';

@Component({
  selector: 'app-lifecycle-demo',
  standalone: true,
  template: `<p>{{ message() }}</p>`
})
export class LifecycleDemoComponent
  implements OnInit, OnChanges, AfterViewInit, OnDestroy {

  message = input<string>('');
  counter = signal(0);

  private subscription?: Subscription;

  constructor() {
    console.log('1. Constructor - Component instance created');
    // Only use for dependency injection
  }

  ngOnChanges(changes: SimpleChanges) {
    console.log('2. OnChanges - Input changed:', changes);
    // React to input property changes
  }

  ngOnInit() {
    console.log('3. OnInit - Component initialized');
    // Fetch data, setup subscriptions
    this.subscription = interval(1000).subscribe(() => {
      this.counter.update(c => c + 1);
    });
  }

  ngAfterViewInit() {
    console.log('5. AfterViewInit - View initialized');
    // Access ViewChild elements here
  }

  ngOnDestroy() {
    console.log('6. OnDestroy - Component destroyed');
    // Cleanup subscriptions, timers
    this.subscription?.unsubscribe();
  }
}
```

### Lifecycle Order

```
1. constructor()
2. ngOnChanges() (if inputs exist)
3. ngOnInit()
4. ngDoCheck()
5. ngAfterContentInit()
6. ngAfterContentChecked()
7. ngAfterViewInit()
8. ngAfterViewChecked()
... (change detection cycles repeat 4-8)
9. ngOnDestroy()
```

## Content Projection

Content projection allows you to insert content from a parent into a child component.

### Single Slot Projection

**Child component:**

```typescript
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="card">
      <ng-content />
    </div>
  `,
  styles: `
    .card {
      border: 1px solid #ddd;
      padding: 1rem;
      border-radius: 8px;
    }
  `
})
export class CardComponent {}
```

**Parent usage:**

```html
<app-card>
  <h3>Card Title</h3>
  <p>Card content goes here</p>
</app-card>
```

### Multi-Slot Projection

```typescript
@Component({
  selector: 'app-dialog',
  standalone: true,
  template: `
    <div class="dialog">
      <header class="dialog-header">
        <ng-content select="[dialog-header]" />
      </header>
      <main class="dialog-body">
        <ng-content />
      </main>
      <footer class="dialog-footer">
        <ng-content select="[dialog-footer]" />
      </footer>
    </div>
  `
})
export class DialogComponent {}
```

**Usage:**

```html
<app-dialog>
  <div dialog-header>
    <h2>Confirm Action</h2>
  </div>

  <p>Are you sure you want to proceed?</p>
  <p>This action cannot be undone.</p>

  <div dialog-footer>
    <button (click)="cancel()">Cancel</button>
    <button (click)="confirm()">Confirm</button>
  </div>
</app-dialog>
```

### Content Selection Options

| Selector | Matches |
|----------|---------|
| `select="header"` | `<header>` elements |
| `select=".card"` | Elements with `card` class |
| `select="[slot]"` | Elements with `slot` attribute |
| `select="[slot=footer]"` | Attribute with specific value |
| `select="app-title"` | Component elements |

### Conditional Content with @if

```typescript
@Component({
  selector: 'app-expandable',
  template: `
    <div class="header" (click)="toggle()">
      <ng-content select="[header]" />
      <span>{{ expanded() ? '▼' : '▶' }}</span>
    </div>
    @if (expanded()) {
      <div class="content">
        <ng-content />
      </div>
    }
  `
})
export class ExpandableComponent {
  expanded = signal(false);

  toggle() {
    this.expanded.update(v => !v);
  }
}
```

## View Queries

Access child elements and components from the parent.

### viewChild

```typescript
import { Component, viewChild, ElementRef, AfterViewInit } from '@angular/core';
import { ChildComponent } from './child.component';

@Component({
  selector: 'app-parent',
  standalone: true,
  imports: [ChildComponent],
  template: `
    <input #inputRef type="text" />
    <app-child #childRef />
  `
})
export class ParentComponent implements AfterViewInit {
  // Query element by template reference
  inputEl = viewChild<ElementRef>('inputRef');

  // Query component
  child = viewChild(ChildComponent);

  // Required query (throws if not found)
  requiredChild = viewChild.required(ChildComponent);

  ngAfterViewInit() {
    // Access after view is initialized
    this.inputEl()?.nativeElement.focus();
    this.child()?.someMethod();
  }
}
```

### viewChildren

```typescript
import { Component, viewChildren } from '@angular/core';
import { ListItemComponent } from './list-item.component';

@Component({
  selector: 'app-list',
  standalone: true,
  imports: [ListItemComponent],
  template: `
    @for (item of items(); track item.id) {
      <app-list-item [data]="item" />
    }
    <button (click)="selectAll()">Select All</button>
  `
})
export class ListComponent {
  items = signal([/* ... */]);

  // Query all matching components
  listItems = viewChildren(ListItemComponent);

  selectAll() {
    // Returns a signal of array
    this.listItems().forEach(item => item.select());
  }
}
```

### contentChild and contentChildren

Query projected content:

```typescript
import { Component, contentChild, contentChildren, AfterContentInit } from '@angular/core';
import { TabComponent } from './tab.component';

@Component({
  selector: 'app-tabs',
  standalone: true,
  template: `
    <div class="tab-headers">
      @for (tab of tabs(); track tab) {
        <button
          [class.active]="tab === activeTab()"
          (click)="selectTab(tab)"
        >
          {{ tab.title() }}
        </button>
      }
    </div>
    <div class="tab-content">
      <ng-content />
    </div>
  `
})
export class TabsComponent implements AfterContentInit {
  // Query projected tab components
  tabs = contentChildren(TabComponent);
  activeTab = contentChild(TabComponent);

  ngAfterContentInit() {
    // Set first tab as active
    const firstTab = this.tabs()[0];
    if (firstTab) {
      firstTab.active.set(true);
    }
  }

  selectTab(tab: TabComponent) {
    this.tabs().forEach(t => t.active.set(false));
    tab.active.set(true);
  }
}
```

## View Encapsulation

Control how component styles are applied.

```typescript
import { Component, ViewEncapsulation } from '@angular/core';

@Component({
  selector: 'app-styled',
  encapsulation: ViewEncapsulation.Emulated, // Default
  template: `<p>Styled content</p>`,
  styles: `p { color: blue; }`
})
export class StyledComponent {}
```

| Mode | Description | Use Case |
|------|-------------|----------|
| `Emulated` | Default. Styles scoped via attribute | Most cases |
| `None` | Styles apply globally | Theme overrides |
| `ShadowDom` | Uses native Shadow DOM | Web components |

### Special Selectors

```scss
// :host - styles the component's host element
:host {
  display: block;
  padding: 1rem;
}

// :host with condition
:host(.active) {
  background: #e3f2fd;
}

// :host-context - styles based on ancestor
:host-context(.dark-theme) {
  background: #333;
  color: white;
}

// ::ng-deep - pierce encapsulation (use sparingly)
:host ::ng-deep .third-party-class {
  color: red;
}
```

## Change Detection

Control when Angular checks for changes.

### Default Change Detection

Angular checks all components on every change detection cycle.

### OnPush Strategy

```typescript
import { Component, ChangeDetectionStrategy, signal } from '@angular/core';

@Component({
  selector: 'app-optimized',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{ data().name }}</p>
    <p>Count: {{ count() }}</p>
  `
})
export class OptimizedComponent {
  data = input.required<{ name: string }>();
  count = signal(0);
}
```

**OnPush triggers change detection when:**

| Trigger | Description |
|---------|-------------|
| Input reference changes | New object/array reference |
| Signal value changes | Any signal read in template |
| Event handler is called | Click, input, etc. |
| Async pipe emits | Observable subscription |
| Manual trigger | `markForCheck()` or `detectChanges()` |

### Manual Change Detection

```typescript
import { Component, ChangeDetectorRef, inject } from '@angular/core';

@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ data }}</p>`
})
export class ManualComponent {
  private cdr = inject(ChangeDetectorRef);
  data = '';

  updateData(newData: string) {
    this.data = newData;
    this.cdr.markForCheck(); // Schedule check
    // or
    this.cdr.detectChanges(); // Check immediately
  }
}
```

## Host Element

Interact with the component's host element.

### host Property

```typescript
@Component({
  selector: 'app-button',
  standalone: true,
  host: {
    'class': 'btn',
    '[class.primary]': 'primary()',
    '[attr.disabled]': 'disabled() || null',
    '[attr.aria-label]': 'label()',
    '(click)': 'onClick($event)'
  },
  template: `<ng-content />`
})
export class ButtonComponent {
  primary = input(false);
  disabled = input(false);
  label = input<string>();

  onClick(event: MouseEvent) {
    if (this.disabled()) {
      event.preventDefault();
    }
  }
}
```

### @HostBinding and @HostListener

```typescript
import { Component, HostBinding, HostListener } from '@angular/core';

@Component({
  selector: 'app-highlight',
  template: `<ng-content />`
})
export class HighlightComponent {
  @HostBinding('class.highlighted')
  isHighlighted = false;

  @HostBinding('style.backgroundColor')
  bgColor = 'transparent';

  @HostListener('mouseenter')
  onMouseEnter() {
    this.isHighlighted = true;
    this.bgColor = 'yellow';
  }

  @HostListener('mouseleave')
  onMouseLeave() {
    this.isHighlighted = false;
    this.bgColor = 'transparent';
  }
}
```

## Summary

| Concept | Description |
|---------|-------------|
| `@Component` | Decorator defining component metadata |
| `input()` | Signal-based inputs (v21) |
| `output()` | Signal-based outputs (v21) |
| `model()` | Two-way binding (v21) |
| Lifecycle Hooks | `ngOnInit`, `ngOnDestroy`, etc. |
| Content Projection | `<ng-content>` for inserting content |
| View Queries | `viewChild`, `viewChildren` |
| Change Detection | `OnPush` for optimization |

## Next Steps

- [Templates & Data Binding](./04-templates-data-binding.md) - Master template syntax
- [Directives](./05-directives.md) - Extend HTML behavior

---

[Previous: Getting Started](./02-getting-started.md) | [Back to Index](./README.md) | [Next: Templates](./04-templates-data-binding.md)
