# Directives

Directives are classes that add behavior to elements in Angular applications. There are three types: components, attribute directives, and structural directives.

## Types of Directives

| Type | Description | Example |
|------|-------------|---------|
| **Components** | Directives with templates | `@Component` |
| **Attribute** | Change appearance/behavior | `ngClass`, `ngStyle` |
| **Structural** | Change DOM structure | `*ngIf`, `*ngFor`, `@if`, `@for` |

## Built-in Attribute Directives

### NgClass

Dynamically add/remove CSS classes:

```typescript
import { Component, signal, computed } from '@angular/core';
import { NgClass } from '@angular/common';

@Component({
  selector: 'app-example',
  standalone: true,
  imports: [NgClass],
  template: `
    <!-- Object syntax -->
    <div [ngClass]="{
      'active': isActive(),
      'disabled': isDisabled(),
      'highlight': isHighlighted()
    }">
      Content
    </div>

    <!-- Array syntax -->
    <div [ngClass]="['class1', 'class2', condition ? 'class3' : '']">
      Content
    </div>

    <!-- String syntax -->
    <div [ngClass]="'class1 class2'">
      Content
    </div>

    <!-- Expression -->
    <div [ngClass]="currentClasses()">
      Content
    </div>
  `
})
export class ExampleComponent {
  isActive = signal(true);
  isDisabled = signal(false);
  isHighlighted = signal(false);

  currentClasses = computed(() => ({
    'active': this.isActive(),
    'disabled': this.isDisabled()
  }));
}
```

### NgStyle

Dynamically set inline styles:

```typescript
import { Component, signal, computed } from '@angular/core';
import { NgStyle } from '@angular/common';

@Component({
  imports: [NgStyle],
  template: `
    <!-- Object syntax -->
    <div [ngStyle]="{
      'color': textColor(),
      'font-size.px': fontSize(),
      'background-color': bgColor()
    }">
      Styled content
    </div>

    <!-- Expression -->
    <div [ngStyle]="currentStyles()">
      Content
    </div>
  `
})
export class StyleComponent {
  textColor = signal('blue');
  fontSize = signal(16);
  bgColor = signal('white');

  currentStyles = computed(() => ({
    'font-weight': this.isImportant() ? 'bold' : 'normal',
    'font-style': this.isItalic() ? 'italic' : 'normal'
  }));
}
```

### NgModel

Two-way data binding for form elements:

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  imports: [FormsModule],
  template: `
    <input [(ngModel)]="username" />
    <input [(ngModel)]="email" type="email" />
    <textarea [(ngModel)]="message"></textarea>

    <select [(ngModel)]="selectedOption">
      @for (option of options; track option.value) {
        <option [value]="option.value">{{ option.label }}</option>
      }
    </select>

    <input type="checkbox" [(ngModel)]="isChecked" />
    <input type="radio" [(ngModel)]="choice" value="a" />
    <input type="radio" [(ngModel)]="choice" value="b" />
  `
})
export class FormComponent {
  username = '';
  email = '';
  message = '';
  selectedOption = '';
  isChecked = false;
  choice = 'a';
  options = [
    { value: '1', label: 'Option 1' },
    { value: '2', label: 'Option 2' }
  ];
}
```

## Built-in Structural Directives

### *ngIf (Legacy)

```typescript
import { Component } from '@angular/core';
import { AsyncPipe, NgIf } from '@angular/common';

@Component({
  imports: [NgIf, AsyncPipe],
  template: `
    <!-- Basic -->
    <div *ngIf="isVisible">Visible content</div>

    <!-- With else -->
    <div *ngIf="isLoggedIn; else loginTemplate">
      Welcome, {{ username }}
    </div>
    <ng-template #loginTemplate>
      <p>Please log in</p>
    </ng-template>

    <!-- With then and else -->
    <ng-container *ngIf="data; then dataTemplate; else loadingTemplate"></ng-container>
    <ng-template #dataTemplate>
      <p>{{ data }}</p>
    </ng-template>
    <ng-template #loadingTemplate>
      <p>Loading...</p>
    </ng-template>

    <!-- With alias -->
    <div *ngIf="user$ | async as user">
      {{ user.name }}
    </div>
  `
})
```

### *ngFor (Legacy)

```typescript
import { Component } from '@angular/core';
import { NgFor } from '@angular/common';

@Component({
  imports: [NgFor],
  template: `
    <!-- Basic -->
    <ul>
      <li *ngFor="let item of items">{{ item.name }}</li>
    </ul>

    <!-- With index -->
    <ul>
      <li *ngFor="let item of items; let i = index">
        {{ i + 1 }}. {{ item.name }}
      </li>
    </ul>

    <!-- With all variables -->
    <ul>
      <li *ngFor="let item of items; let i = index; let first = first; let last = last; let even = even; let odd = odd"
          [class.first]="first"
          [class.last]="last"
          [class.even]="even"
          [class.odd]="odd">
        {{ item.name }}
      </li>
    </ul>

    <!-- With trackBy -->
    <ul>
      <li *ngFor="let item of items; trackBy: trackByFn">
        {{ item.name }}
      </li>
    </ul>
  `
})
export class ListComponent {
  items = [/* ... */];

  trackByFn(index: number, item: any) {
    return item.id;
  }
}
```

### *ngSwitch (Legacy)

```typescript
import { Component } from '@angular/core';
import { NgSwitch, NgSwitchCase, NgSwitchDefault } from '@angular/common';

@Component({
  imports: [NgSwitch, NgSwitchCase, NgSwitchDefault],
  template: `
    <div [ngSwitch]="status">
      <p *ngSwitchCase="'loading'">Loading...</p>
      <p *ngSwitchCase="'success'">Success!</p>
      <p *ngSwitchCase="'error'">Error occurred</p>
      <p *ngSwitchDefault>Unknown status</p>
    </div>
  `
})
```

## Modern Control Flow (v21+)

Prefer built-in control flow over structural directives:

```html
<!-- Instead of *ngIf -->
@if (isVisible()) {
  <div>Visible</div>
}

<!-- Instead of *ngFor -->
@for (item of items(); track item.id) {
  <div>{{ item.name }}</div>
}

<!-- Instead of *ngSwitch -->
@switch (status()) {
  @case ('loading') { <p>Loading...</p> }
  @case ('success') { <p>Success!</p> }
  @default { <p>Unknown</p> }
}
```

## Creating Custom Attribute Directives

### Generate Directive

```bash
ng generate directive highlight
# or
ng g d highlight
```

### Basic Directive

```typescript
import { Directive, ElementRef, inject } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  private el = inject(ElementRef);

  constructor() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

**Usage:**

```html
<p appHighlight>This text is highlighted</p>
```

### Directive with Input

```typescript
import { Directive, ElementRef, input, effect, inject } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  private el = inject(ElementRef);

  // Input with same name as selector
  appHighlight = input<string>('yellow');

  // Alternative color
  defaultColor = input<string>('');

  constructor() {
    effect(() => {
      const color = this.appHighlight() || this.defaultColor() || 'yellow';
      this.el.nativeElement.style.backgroundColor = color;
    });
  }
}
```

**Usage:**

```html
<p appHighlight>Yellow (default)</p>
<p [appHighlight]="'lightblue'">Light blue</p>
<p appHighlight="pink">Pink</p>
<p appHighlight [defaultColor]="'lightgreen'">Green (from default)</p>
```

### Directive with Host Listeners

```typescript
import { Directive, ElementRef, HostListener, inject, signal } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  private el = inject(ElementRef);
  private isHighlighted = signal(false);

  @HostListener('mouseenter')
  onMouseEnter() {
    this.highlight('yellow');
    this.isHighlighted.set(true);
  }

  @HostListener('mouseleave')
  onMouseLeave() {
    this.highlight('');
    this.isHighlighted.set(false);
  }

  @HostListener('click')
  onClick() {
    console.log('Element clicked, highlighted:', this.isHighlighted());
  }

  private highlight(color: string) {
    this.el.nativeElement.style.backgroundColor = color;
  }
}
```

### Modern Host Binding (v21)

```typescript
import { Directive, input, signal } from '@angular/core';

@Directive({
  selector: '[appTooltip]',
  standalone: true,
  host: {
    '[class.has-tooltip]': 'true',
    '[attr.data-tooltip]': 'appTooltip()',
    '(mouseenter)': 'show()',
    '(mouseleave)': 'hide()'
  }
})
export class TooltipDirective {
  appTooltip = input.required<string>();

  private visible = signal(false);

  show() {
    this.visible.set(true);
  }

  hide() {
    this.visible.set(false);
  }
}
```

### Directive with Renderer2

For safe DOM manipulation:

```typescript
import { Directive, ElementRef, Renderer2, inject, OnInit } from '@angular/core';

@Directive({
  selector: '[appSafeStyle]',
  standalone: true
})
export class SafeStyleDirective implements OnInit {
  private el = inject(ElementRef);
  private renderer = inject(Renderer2);

  ngOnInit() {
    // Safe DOM manipulation
    this.renderer.setStyle(this.el.nativeElement, 'border', '1px solid blue');
    this.renderer.addClass(this.el.nativeElement, 'styled');
    this.renderer.setAttribute(this.el.nativeElement, 'data-styled', 'true');
  }
}
```

## Creating Custom Structural Directives

### Basic Structural Directive

```typescript
import { Directive, TemplateRef, ViewContainerRef, input, effect, inject } from '@angular/core';

@Directive({
  selector: '[appIf]',
  standalone: true
})
export class IfDirective {
  private templateRef = inject(TemplateRef<any>);
  private viewContainer = inject(ViewContainerRef);

  appIf = input.required<boolean>();

  private hasView = false;

  constructor() {
    effect(() => {
      if (this.appIf() && !this.hasView) {
        this.viewContainer.createEmbeddedView(this.templateRef);
        this.hasView = true;
      } else if (!this.appIf() && this.hasView) {
        this.viewContainer.clear();
        this.hasView = false;
      }
    });
  }
}
```

**Usage:**

```html
<div *appIf="isVisible()">Conditionally shown</div>
```

### Repeat Directive

```typescript
import { Directive, TemplateRef, ViewContainerRef, input, effect, inject } from '@angular/core';

@Directive({
  selector: '[appRepeat]',
  standalone: true
})
export class RepeatDirective {
  private templateRef = inject(TemplateRef<any>);
  private viewContainer = inject(ViewContainerRef);

  appRepeat = input.required<number>();

  constructor() {
    effect(() => {
      this.viewContainer.clear();

      for (let i = 0; i < this.appRepeat(); i++) {
        this.viewContainer.createEmbeddedView(this.templateRef, {
          $implicit: i,
          index: i,
          first: i === 0,
          last: i === this.appRepeat() - 1
        });
      }
    });
  }
}
```

**Usage:**

```html
<div *appRepeat="5; let i = index; let first = first">
  Item {{ i + 1 }} {{ first ? '(First!)' : '' }}
</div>
```

### Permission Directive

```typescript
import { Directive, TemplateRef, ViewContainerRef, input, inject, effect } from '@angular/core';
import { AuthService } from './auth.service';

@Directive({
  selector: '[appHasPermission]',
  standalone: true
})
export class HasPermissionDirective {
  private templateRef = inject(TemplateRef<any>);
  private viewContainer = inject(ViewContainerRef);
  private authService = inject(AuthService);

  appHasPermission = input.required<string | string[]>();

  private hasView = false;

  constructor() {
    effect(() => {
      const permissions = this.appHasPermission();
      const permissionList = Array.isArray(permissions) ? permissions : [permissions];

      const hasPermission = permissionList.some(p =>
        this.authService.hasPermission(p)
      );

      if (hasPermission && !this.hasView) {
        this.viewContainer.createEmbeddedView(this.templateRef);
        this.hasView = true;
      } else if (!hasPermission && this.hasView) {
        this.viewContainer.clear();
        this.hasView = false;
      }
    });
  }
}
```

**Usage:**

```html
<button *appHasPermission="'admin'">Admin Only</button>
<button *appHasPermission="['admin', 'editor']">Admin or Editor</button>
```

## Directive Composition

Combine multiple directives:

```typescript
import { Directive, input } from '@angular/core';
import { HighlightDirective } from './highlight.directive';
import { TooltipDirective } from './tooltip.directive';

@Directive({
  selector: '[appCard]',
  standalone: true,
  hostDirectives: [
    {
      directive: HighlightDirective,
      inputs: ['appHighlight: cardColor']
    },
    {
      directive: TooltipDirective,
      inputs: ['appTooltip: cardTooltip']
    }
  ]
})
export class CardDirective {
  // Additional card-specific logic
}
```

**Usage:**

```html
<div appCard cardColor="lightblue" cardTooltip="This is a card">
  Card content
</div>
```

## Directive with Output

```typescript
import { Directive, output, HostListener } from '@angular/core';

@Directive({
  selector: '[appClickOutside]',
  standalone: true
})
export class ClickOutsideDirective {
  clickOutside = output<void>();

  @HostListener('document:click', ['$event.target'])
  onClick(target: HTMLElement) {
    // Logic to detect click outside
    this.clickOutside.emit();
  }
}
```

## Common Directive Patterns

### Auto Focus

```typescript
import { inject, Directive, ElementRef, AfterViewInit } from '@angular/core';
@Directive({
  selector: '[appAutoFocus]',
  standalone: true
})
export class AutoFocusDirective implements AfterViewInit {
  private el = inject(ElementRef);

  ngAfterViewInit() {
    this.el.nativeElement.focus();
  }
}
```

### Debounce Click

```typescript
import { input, Directive, HostListener, output } from '@angular/core';
import { debounceTime } from 'rxjs/operators';
import { Subject } from 'rxjs';
@Directive({
  selector: '[appDebounceClick]',
  standalone: true
})
export class DebounceClickDirective {
  private clicks = new Subject<MouseEvent>();

  debounceTime = input(300);
  debounceClick = output<MouseEvent>();

  constructor() {
    this.clicks.pipe(
      debounceTime(this.debounceTime()),
      takeUntilDestroyed()
    ).subscribe(e => this.debounceClick.emit(e));
  }

  @HostListener('click', ['$event'])
  onClick(event: MouseEvent) {
    event.preventDefault();
    event.stopPropagation();
    this.clicks.next(event);
  }
}
```

### Lazy Load Image

```typescript
import { inject, Directive, ElementRef, OnInit, OnDestroy } from '@angular/core';
@Directive({
  selector: 'img[appLazyLoad]',
  standalone: true
})
export class LazyLoadDirective implements OnInit, OnDestroy {
  private el = inject(ElementRef);

  appLazyLoad = input.required<string>();
  private observer?: IntersectionObserver;

  ngOnInit() {
    this.observer = new IntersectionObserver(entries => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          this.el.nativeElement.src = this.appLazyLoad();
          this.observer?.disconnect();
        }
      });
    });
    this.observer.observe(this.el.nativeElement);
  }

  ngOnDestroy() {
    this.observer?.disconnect();
  }
}
```

## Summary

| Type | Purpose | Example |
|------|---------|---------|
| Attribute | Modify appearance/behavior | `appHighlight`, `ngClass` |
| Structural | Change DOM structure | `*ngIf`, `@if`, `*ngFor`, `@for` |
| Host | Bind to host element | `host: {}`, `@HostBinding` |
| Composition | Combine directives | `hostDirectives` |

## Next Steps

- [Pipes](./06-pipes.md) - Transform data in templates
- [Services & DI](./07-services-and-dependency-injection.md) - Share logic across components

---

[Previous: Templates](./04-templates-data-binding.md) | [Back to Index](./README.md) | [Next: Pipes](./06-pipes.md)
