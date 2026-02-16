# Pipes

Pipes transform data for display in templates. They take an input value and return a transformed value.

## Syntax

```html
{{ value | pipeName }}
{{ value | pipeName:arg1:arg2 }}
{{ value | pipe1 | pipe2 | pipe3 }}
```

## Built-in Pipes

### DatePipe

Format dates:

```typescript
import { Component } from '@angular/core';
import { DatePipe } from '@angular/common';

@Component({
  standalone: true,
  imports: [DatePipe],
  template: `
    <p>{{ today | date }}</p>
    <p>{{ today | date:'short' }}</p>
    <p>{{ today | date:'medium' }}</p>
    <p>{{ today | date:'long' }}</p>
    <p>{{ today | date:'full' }}</p>
    <p>{{ today | date:'shortDate' }}</p>
    <p>{{ today | date:'longDate' }}</p>
    <p>{{ today | date:'shortTime' }}</p>
    <p>{{ today | date:'longTime' }}</p>
    <p>{{ today | date:'yyyy-MM-dd' }}</p>
    <p>{{ today | date:'HH:mm:ss' }}</p>
    <p>{{ today | date:'EEEE, MMMM d, y' }}</p>
  `
})
export class DateComponent {
  today = new Date();
}
```

**Format options:**

| Format | Example |
|--------|---------|
| `short` | 1/25/26, 3:30 PM |
| `medium` | Jan 25, 2026, 3:30:00 PM |
| `long` | January 25, 2026 at 3:30:00 PM GMT |
| `full` | Saturday, January 25, 2026 at 3:30:00 PM GMT |
| `shortDate` | 1/25/26 |
| `mediumDate` | Jan 25, 2026 |
| `longDate` | January 25, 2026 |
| `fullDate` | Saturday, January 25, 2026 |
| `shortTime` | 3:30 PM |
| `mediumTime` | 3:30:00 PM |

**Custom format characters:**

| Character | Meaning |
|-----------|---------|
| `y` | Year |
| `M` | Month |
| `d` | Day |
| `E` | Day of week |
| `H` | Hour (0-23) |
| `h` | Hour (1-12) |
| `m` | Minute |
| `s` | Second |
| `a` | AM/PM |

### CurrencyPipe

Format currency:

```typescript
import { Component } from '@angular/core';
import { CurrencyPipe } from '@angular/common';

@Component({
  imports: [CurrencyPipe],
  template: `
    <p>{{ price | currency }}</p>
    <p>{{ price | currency:'EUR' }}</p>
    <p>{{ price | currency:'GBP':'symbol' }}</p>
    <p>{{ price | currency:'USD':'symbol':'1.0-0' }}</p>
    <p>{{ price | currency:'JPY':'symbol-narrow' }}</p>
  `
})
export class PriceComponent {
  price = 1234.56;
}
```

**Arguments:**

```
{{ value | currency:currencyCode:display:digitsInfo:locale }}
```

| Display | Description |
|---------|-------------|
| `code` | USD |
| `symbol` | $ |
| `symbol-narrow` | Narrow symbol |

### DecimalPipe

Format numbers:

```typescript
import { Component } from '@angular/core';
import { DecimalPipe } from '@angular/common';

@Component({
  imports: [DecimalPipe],
  template: `
    <p>{{ number | number }}</p>
    <p>{{ number | number:'1.2-2' }}</p>
    <p>{{ number | number:'4.0-0' }}</p>
    <p>{{ pi | number:'1.0-4' }}</p>
  `
})
export class NumberComponent {
  number = 1234.5678;
  pi = 3.14159265359;
}
```

**Format:** `{minIntegerDigits}.{minFractionDigits}-{maxFractionDigits}`

| Example | Input | Output |
|---------|-------|--------|
| `1.2-2` | 3.1 | 3.10 |
| `1.2-2` | 3.1415 | 3.14 |
| `4.0-0` | 42 | 0,042 |

### PercentPipe

Format percentages:

```typescript
import { Component } from '@angular/core';
import { PercentPipe } from '@angular/common';

@Component({
  imports: [PercentPipe],
  template: `
    <p>{{ ratio | percent }}</p>
    <p>{{ ratio | percent:'1.2-2' }}</p>
  `
})
export class PercentComponent {
  ratio = 0.856; // 85.6%
}
```

### UpperCasePipe / LowerCasePipe / TitleCasePipe

```typescript
import { Component } from '@angular/core';
import { UpperCasePipe, LowerCasePipe, TitleCasePipe } from '@angular/common';

@Component({
  imports: [UpperCasePipe, LowerCasePipe, TitleCasePipe],
  template: `
    <p>{{ name | uppercase }}</p>  <!-- JOHN DOE -->
    <p>{{ name | lowercase }}</p>  <!-- john doe -->
    <p>{{ name | titlecase }}</p>  <!-- John Doe -->
  `
})
export class TextComponent {
  name = 'john DOE';
}
```

### SlicePipe

Slice arrays or strings:

```typescript
import { Component } from '@angular/core';
import { SlicePipe } from '@angular/common';

@Component({
  imports: [SlicePipe],
  template: `
    <!-- Arrays -->
    @for (item of items | slice:0:5; track item) {
      <p>{{ item }}</p>
    }

    <!-- Strings -->
    <p>{{ longText | slice:0:100 }}...</p>
  `
})
export class SliceComponent {
  items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  longText = 'Lorem ipsum dolor sit amet...';
}
```

### JsonPipe

Debug by displaying JSON:

```typescript
import { Component } from '@angular/core';
import { JsonPipe } from '@angular/common';

@Component({
  imports: [JsonPipe],
  template: `
    <pre>{{ data | json }}</pre>
  `
})
export class DebugComponent {
  data = { name: 'John', age: 30, roles: ['admin', 'user'] };
}
```

### KeyValuePipe

Iterate over objects:

```typescript
import { Component } from '@angular/core';
import { KeyValuePipe } from '@angular/common';

@Component({
  imports: [KeyValuePipe],
  template: `
    @for (item of user | keyvalue; track item.key) {
      <p>{{ item.key }}: {{ item.value }}</p>
    }

    <!-- With custom comparator -->
    @for (item of user | keyvalue:sortByKey; track item.key) {
      <p>{{ item.key }}: {{ item.value }}</p>
    }
  `
})
export class ObjectComponent {
  user = { name: 'John', age: 30, city: 'NYC' };

  sortByKey(a: any, b: any) {
    return a.key.localeCompare(b.key);
  }
}
```

### AsyncPipe

Subscribe to Observables/Promises:

```typescript
import { Component } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { Observable, interval } from 'rxjs';
import { map } from 'rxjs/operators';

@Component({
  imports: [AsyncPipe],
  template: `
    <!-- Observable -->
    <p>Time: {{ time$ | async }}</p>

    <!-- With @if -->
    @if (user$ | async; as user) {
      <p>Welcome, {{ user.name }}</p>
    }

    <!-- With @for -->
    @for (item of items$ | async; track item.id) {
      <p>{{ item.name }}</p>
    }

    <!-- Promise -->
    <p>{{ dataPromise | async }}</p>
  `
})
export class AsyncComponent {
  time$ = interval(1000).pipe(
    map(() => new Date().toLocaleTimeString())
  );

  user$: Observable<User>;
  items$: Observable<Item[]>;
  dataPromise: Promise<string>;
}
```

**Benefits:**

- Automatically subscribes/unsubscribes
- Works with change detection
- Prevents memory leaks

## Creating Custom Pipes

### Generate Pipe

```bash
ng generate pipe truncate
# or
ng g p truncate
```

### Simple Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'truncate',
  standalone: true
})
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50, ellipsis: string = '...'): string {
    if (!value) return '';
    if (value.length <= limit) return value;
    return value.substring(0, limit) + ellipsis;
  }
}
```

**Usage:**

```html
<p>{{ longText | truncate }}</p>
<p>{{ longText | truncate:100 }}</p>
<p>{{ longText | truncate:100:'[...]' }}</p>
```

### Filter Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'filter',
  standalone: true
})
export class FilterPipe implements PipeTransform {
  transform<T>(items: T[], field: keyof T, value: any): T[] {
    if (!items || !field || value === undefined) {
      return items;
    }
    return items.filter(item => item[field] === value);
  }
}
```

**Usage:**

```html
@for (product of products | filter:'category':'electronics'; track product.id) {
  <p>{{ product.name }}</p>
}
```

### Search Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'search',
  standalone: true
})
export class SearchPipe implements PipeTransform {
  transform<T>(items: T[], searchTerm: string, fields: (keyof T)[]): T[] {
    if (!items || !searchTerm || !fields.length) {
      return items;
    }

    const term = searchTerm.toLowerCase();

    return items.filter(item =>
      fields.some(field => {
        const value = item[field];
        return value?.toString().toLowerCase().includes(term);
      })
    );
  }
}
```

**Usage:**

```html
@for (user of users | search:searchTerm:['name', 'email']; track user.id) {
  <p>{{ user.name }} - {{ user.email }}</p>
}
```

### Sort Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'sort',
  standalone: true
})
export class SortPipe implements PipeTransform {
  transform<T>(items: T[], field: keyof T, direction: 'asc' | 'desc' = 'asc'): T[] {
    if (!items || !field) {
      return items;
    }

    return [...items].sort((a, b) => {
      const aVal = a[field];
      const bVal = b[field];

      if (aVal < bVal) return direction === 'asc' ? -1 : 1;
      if (aVal > bVal) return direction === 'asc' ? 1 : -1;
      return 0;
    });
  }
}
```

**Usage:**

```html
@for (product of products | sort:'price':'asc'; track product.id) {
  <p>{{ product.name }} - {{ product.price | currency }}</p>
}
```

### Highlight Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
import { DomSanitizer, SafeHtml } from '@angular/platform-browser';

@Pipe({
  name: 'highlight',
  standalone: true
})
export class HighlightPipe implements PipeTransform {
  constructor(private sanitizer: DomSanitizer) {}

  transform(text: string, search: string): SafeHtml {
    if (!search || !text) {
      return text;
    }

    const regex = new RegExp(`(${this.escapeRegex(search)})`, 'gi');
    const highlighted = text.replace(regex, '<mark>$1</mark>');

    return this.sanitizer.bypassSecurityTrustHtml(highlighted);
  }

  private escapeRegex(str: string): string {
    return str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  }
}
```

**Usage:**

```html
<p [innerHTML]="text | highlight:searchTerm"></p>
```

### Time Ago Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'timeAgo',
  standalone: true
})
export class TimeAgoPipe implements PipeTransform {
  transform(value: Date | string | number): string {
    const date = new Date(value);
    const now = new Date();
    const seconds = Math.floor((now.getTime() - date.getTime()) / 1000);

    const intervals: [number, string][] = [
      [31536000, 'year'],
      [2592000, 'month'],
      [86400, 'day'],
      [3600, 'hour'],
      [60, 'minute'],
      [1, 'second']
    ];

    for (const [secondsInInterval, name] of intervals) {
      const count = Math.floor(seconds / secondsInInterval);
      if (count >= 1) {
        return count === 1
          ? `1 ${name} ago`
          : `${count} ${name}s ago`;
      }
    }

    return 'just now';
  }
}
```

**Usage:**

```html
<p>Posted {{ post.createdAt | timeAgo }}</p>
<!-- "2 hours ago", "3 days ago", etc. -->
```

### File Size Pipe

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'fileSize',
  standalone: true
})
export class FileSizePipe implements PipeTransform {
  private units = ['B', 'KB', 'MB', 'GB', 'TB'];

  transform(bytes: number, decimals: number = 2): string {
    if (bytes === 0) return '0 B';

    const k = 1024;
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    const size = bytes / Math.pow(k, i);

    return `${size.toFixed(decimals)} ${this.units[i]}`;
  }
}
```

**Usage:**

```html
<p>File size: {{ file.size | fileSize }}</p>
<p>File size: {{ file.size | fileSize:0 }}</p>
```

## Pure vs Impure Pipes

### Pure Pipes (Default)

- Only re-evaluated when input reference changes
- More performant
- Default behavior

```typescript
import { Pipe } from '@angular/core';
@Pipe({
  name: 'myPipe',
  standalone: true,
  pure: true  // Default
})
```

### Impure Pipes

- Re-evaluated on every change detection cycle
- Required when pipe depends on external state
- Use sparingly (performance impact)

```typescript
import { Pipe, PipeTransform } from '@angular/core';
@Pipe({
  name: 'filterByStatus',
  standalone: true,
  pure: false  // Impure
})
export class FilterByStatusPipe implements PipeTransform {
  constructor(private statusService: StatusService) {}

  transform(items: Item[]): Item[] {
    const activeStatus = this.statusService.getActiveStatus();
    return items.filter(item => item.status === activeStatus);
  }
}
```

### When to Use Impure Pipes

- Pipe depends on service state
- Filtering arrays that are mutated
- Pipe uses async operations internally

**Better alternatives:**

```typescript
import { computed } from '@angular/core';
// Instead of impure pipe, use computed signal
filteredItems = computed(() =>
  this.items().filter(item => item.status === this.statusService.activeStatus())
);
```

## Pipe with Dependency Injection

```typescript
import { Pipe, PipeTransform, inject } from '@angular/core';
import { TranslateService } from './translate.service';

@Pipe({
  name: 'translate',
  standalone: true
})
export class TranslatePipe implements PipeTransform {
  private translateService = inject(TranslateService);

  transform(key: string, params?: Record<string, any>): string {
    return this.translateService.translate(key, params);
  }
}
```

## Async Pipe Pattern

Create pipes that handle async operations:

```typescript
import { inject, Pipe, PipeTransform } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { shareReplay } from 'rxjs/operators';
import { Observable } from 'rxjs';
@Pipe({
  name: 'fetch',
  standalone: true
})
export class FetchPipe implements PipeTransform {
  private http = inject(HttpClient);
  private cache = new Map<string, Observable<any>>();

  transform(url: string): Observable<any> {
    if (!this.cache.has(url)) {
      this.cache.set(url, this.http.get(url).pipe(
        shareReplay(1)
      ));
    }
    return this.cache.get(url)!;
  }
}
```

**Usage with async pipe:**

```html
@if ('https://api.example.com/user' | fetch | async; as user) {
  <p>{{ user.name }}</p>
}
```

## Chaining Pipes

```html
<!-- Chain multiple pipes -->
<p>{{ name | lowercase | titlecase }}</p>
<p>{{ price | currency:'USD' | slice:0:10 }}</p>
<p>{{ items | filter:'active' | sort:'name' | slice:0:10 }}</p>

<!-- With async -->
<p>{{ (data$ | async)?.name | uppercase }}</p>

@for (item of items$ | async | filter:'category':selectedCategory | sort:'price'; track item.id) {
  <p>{{ item.name }}</p>
}
```

## Summary

| Pipe | Purpose | Example |
|------|---------|---------|
| `date` | Format dates | `{{ date \| date:'short' }}` |
| `currency` | Format currency | `{{ price \| currency:'USD' }}` |
| `number` | Format numbers | `{{ num \| number:'1.2-2' }}` |
| `percent` | Format percentages | `{{ ratio \| percent }}` |
| `uppercase` | Uppercase text | `{{ text \| uppercase }}` |
| `lowercase` | Lowercase text | `{{ text \| lowercase }}` |
| `titlecase` | Title case text | `{{ text \| titlecase }}` |
| `slice` | Slice array/string | `{{ arr \| slice:0:5 }}` |
| `json` | JSON stringify | `{{ obj \| json }}` |
| `keyvalue` | Object to array | `{{ obj \| keyvalue }}` |
| `async` | Subscribe to Observable | `{{ obs$ \| async }}` |

## Next Steps

- [Services & Dependency Injection](./07-services-and-dependency-injection.md) - Share logic and data
- [Routing & Navigation](./08-routing.md) - Navigate between views

---

[Previous: Directives](./05-directives.md) | [Back to Index](./README.md) | [Next: Services & Dependency Injection](./07-services-and-dependency-injection.md)
