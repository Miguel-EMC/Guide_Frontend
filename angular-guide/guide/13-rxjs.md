# RxJS & Observables

RxJS (Reactive Extensions for JavaScript) is a library for reactive programming using Observables. Angular uses RxJS extensively for async operations.

## Core Concepts

| Concept | Description |
|---------|-------------|
| **Observable** | Lazy push collection of values over time |
| **Observer** | Consumer of observable values |
| **Subscription** | Execution of an observable |
| **Operators** | Pure functions to transform observables |
| **Subject** | Observable that can multicast |

## Creating Observables

### From Scratch

```typescript
import { Observable } from 'rxjs';

const custom$ = new Observable<number>(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

custom$.subscribe({
  next: value => console.log(value),
  error: err => console.error(err),
  complete: () => console.log('Done')
});
```

### Creation Functions

```typescript
import { of, from, interval, timer, fromEvent, throwError, EMPTY, NEVER } from 'rxjs';

// of - emit values
const values$ = of(1, 2, 3, 4, 5);

// from - convert array, promise, iterable
const array$ = from([1, 2, 3]);
const promise$ = from(fetch('/api/data'));

// interval - emit every n milliseconds
const interval$ = interval(1000); // 0, 1, 2, 3...

// timer - emit after delay, optionally repeat
const timer$ = timer(2000); // emit 0 after 2s
const timerInterval$ = timer(2000, 1000); // start after 2s, then every 1s

// fromEvent - DOM events
const clicks$ = fromEvent(document, 'click');

// throwError - emit error immediately
const error$ = throwError(() => new Error('Oops'));

// EMPTY - complete immediately
// NEVER - never emit, never complete
```

## Subscribing

```typescript
// Full observer
observable$.subscribe({
  next: value => console.log(value),
  error: err => console.error(err),
  complete: () => console.log('Complete')
});

// Just next handler
observable$.subscribe(value => console.log(value));

// Unsubscribe
const subscription = observable$.subscribe(console.log);
subscription.unsubscribe();
```

## Essential Operators

### Transformation

```typescript
import { map, scan, reduce } from 'rxjs/operators';
import { of } from 'rxjs';

// map - transform each value
of(1, 2, 3).pipe(
  map(x => x * 2)
); // 2, 4, 6

// scan - accumulator (like reduce but emits each)
of(1, 2, 3).pipe(
  scan((acc, val) => acc + val, 0)
); // 1, 3, 6

// reduce - final accumulated value
of(1, 2, 3).pipe(
  reduce((acc, val) => acc + val, 0)
); // 6
```

### Filtering

```typescript
import { filter, take, takeUntil, first, distinctUntilChanged } from 'rxjs/operators';
import { interval, timer, of } from 'rxjs';

// filter - only matching values
of(1, 2, 3, 4, 5).pipe(
  filter(x => x > 2)
); // 3, 4, 5

// take - first n values then complete
interval(100).pipe(
  take(5)
); // 0, 1, 2, 3, 4

// takeUntil - take until another observable emits
const stop$ = timer(5000);
interval(100).pipe(
  takeUntil(stop$)
);

// distinctUntilChanged - only emit when value changes
of(1, 1, 2, 2, 3, 1).pipe(
  distinctUntilChanged()
); // 1, 2, 3, 1

// first - first value (or first matching)
of(1, 2, 3).pipe(first()); // 1
of(1, 2, 3).pipe(first(x => x > 1)); // 2
```

### Combination

```typescript
import { combineLatest, merge, concat, forkJoin, zip, withLatestFrom, of } from 'rxjs';

const a$ = of(1, 2);
const b$ = of('a', 'b');

// combineLatest - emit when any emits (after all have emitted)
combineLatest([a$, b$]); // [2, 'a'], [2, 'b']

// merge - interleave emissions
merge(a$, b$); // 1, 2, 'a', 'b' (order may vary)

// concat - sequential
concat(a$, b$); // 1, 2, 'a', 'b'

// forkJoin - wait for all to complete, emit last values
forkJoin([a$, b$]); // [2, 'b']

// zip - pair by index
zip(a$, b$); // [1, 'a'], [2, 'b']

// withLatestFrom - combine with latest from another
a$.pipe(
  withLatestFrom(b$)
); // [1, 'b'], [2, 'b']
```

### Flattening

```typescript
import { switchMap, mergeMap, concatMap, exhaustMap } from 'rxjs/operators';

// switchMap - cancel previous, use latest
searchTerm$.pipe(
  switchMap(term => searchApi(term))
);

// mergeMap - run in parallel
clicks$.pipe(
  mergeMap(() => saveData())
);

// concatMap - queue sequentially
queue$.pipe(
  concatMap(item => processItem(item))
);

// exhaustMap - ignore while busy
clicks$.pipe(
  exhaustMap(() => submitForm())
);
```

### Error Handling

```typescript
import { catchError, retry, timer, of } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';

// catchError - handle errors
http.get('/api/data').pipe(
  catchError(error => {
    console.error('Error:', error);
    return of([]); // Return fallback
  })
);

// retry - retry n times
http.get('/api/data').pipe(
  retry(3)
);

// retry with delay
http.get('/api/data').pipe(
  retry({
    count: 3,
    delay: 1000
  })
);

// Exponential backoff
http.get('/api/data').pipe(
  retry({
    count: 3,
    delay: (error, retryCount) => timer(Math.pow(2, retryCount) * 1000)
  })
);
```

### Timing

```typescript
import { debounceTime, throttleTime, delay, timeout } from 'rxjs/operators';
import { of } from 'rxjs';

// debounceTime - wait for pause
searchInput$.pipe(
  debounceTime(300)
);

// throttleTime - emit first, ignore for duration
scroll$.pipe(
  throttleTime(100)
);

// delay - delay emissions
of(1, 2, 3).pipe(
  delay(1000)
);

// timeout - error if no emission in time
http.get('/api/data').pipe(
  timeout(5000)
);
```

### Utility

```typescript
import { tap, finalize, share, shareReplay } from 'rxjs/operators';

// tap - side effects
data$.pipe(
  tap(value => console.log('Got:', value)),
  tap({
    next: v => console.log('Next:', v),
    error: e => console.error('Error:', e),
    complete: () => console.log('Complete')
  })
);

// finalize - run on complete or error
http.get('/api/data').pipe(
  finalize(() => hideLoading())
);

// share - share subscription
const shared$ = data$.pipe(share());

// shareReplay - share and replay last n values
const cached$ = data$.pipe(shareReplay(1));
```

## Subjects

Subjects are both Observable and Observer.

### Subject

```typescript
import { Subject } from 'rxjs';

const subject = new Subject<number>();

// Subscribe
subject.subscribe(v => console.log('A:', v));
subject.subscribe(v => console.log('B:', v));

// Emit
subject.next(1); // A: 1, B: 1
subject.next(2); // A: 2, B: 2
```

### BehaviorSubject

Has current value, emits to new subscribers immediately.

```typescript
import { BehaviorSubject } from 'rxjs';

const subject = new BehaviorSubject<number>(0);

console.log(subject.value); // 0

subject.subscribe(v => console.log('A:', v)); // A: 0

subject.next(1); // A: 1

subject.subscribe(v => console.log('B:', v)); // B: 1

subject.next(2); // A: 2, B: 2
```

### ReplaySubject

Replays last n values to new subscribers.

```typescript
import { ReplaySubject } from 'rxjs';

const subject = new ReplaySubject<number>(2); // buffer 2

subject.next(1);
subject.next(2);
subject.next(3);

subject.subscribe(v => console.log(v)); // 2, 3
```

### AsyncSubject

Only emits last value on complete.

```typescript
import { AsyncSubject } from 'rxjs';

const subject = new AsyncSubject<number>();

subject.subscribe(v => console.log(v));

subject.next(1);
subject.next(2);
subject.next(3);
subject.complete(); // Now emits: 3
```

## Common Patterns

### Search with Debounce

```typescript
import { Component, input, signal } from '@angular/core';
import { switchMap, filter, debounceTime, distinctUntilChanged } from 'rxjs/operators';
import { Subject } from 'rxjs';
@Component({
  template: `
    <input (input)="onSearch($event)" />
    @for (result of results(); track result.id) {
      <div>{{ result.name }}</div>
    }
  `
})
export class SearchComponent {
  private searchTerms = new Subject<string>();
  results = signal<SearchResult[]>([]);

  constructor() {
    this.searchTerms.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(term => term.length >= 2),
      switchMap(term => this.searchService.search(term)),
      takeUntilDestroyed()
    ).subscribe(results => this.results.set(results));
  }

  onSearch(event: Event) {
    const term = (event.target as HTMLInputElement).value;
    this.searchTerms.next(term);
  }
}
```

### Auto-refresh

```typescript
import { Component } from '@angular/core';
import { switchMap, shareReplay } from 'rxjs/operators';
import { Subject, interval, merge } from 'rxjs';
@Component({...})
export class DashboardComponent {
  private refresh$ = new Subject<void>();

  data$ = merge(
    this.refresh$,
    interval(30000) // Auto-refresh every 30s
  ).pipe(
    startWith(null),
    switchMap(() => this.dataService.getData()),
    shareReplay(1)
  );

  refresh() {
    this.refresh$.next();
  }
}
```

### Polling

```typescript
import { switchMap, retry, takeUntil } from 'rxjs/operators';
import { timer } from 'rxjs';
pollData() {
  return timer(0, 5000).pipe(
    switchMap(() => this.http.get<Data>('/api/data')),
    retry(3),
    takeUntil(this.stopPolling$)
  );
}
```

### Race Condition Prevention

```typescript
import { switchMap, exhaustMap } from 'rxjs/operators';
// Cancel previous request when new one starts
loadUser(id: number) {
  return this.userId$.pipe(
    switchMap(id => this.userService.getUser(id))
  );
}

// Or with exhaustMap to ignore during loading
submitForm() {
  this.submit$.pipe(
    exhaustMap(() => this.formService.submit(this.form.value))
  );
}
```

## RxJS with Signals

### toSignal

```typescript
import { Component } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { catchError } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({...})
export class DataComponent {
  private data$ = this.http.get<Data[]>('/api/data');

  // Convert to signal
  data = toSignal(this.data$, { initialValue: [] });

  // With error handling
  dataWithError = toSignal(
    this.data$.pipe(
      catchError(() => of([]))
    ),
    { initialValue: [] }
  );
}
```

### toObservable

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { switchMap, debounceTime, distinctUntilChanged } from 'rxjs/operators';

@Component({...})
export class SearchComponent {
  searchTerm = signal('');

  // Convert signal to observable
  results = toSignal(
    toObservable(this.searchTerm).pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => this.search(term))
    ),
    { initialValue: [] }
  );
}
```

### takeUntilDestroyed

```typescript
import { Component, inject, DestroyRef } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';

@Component({...})
export class AutoCleanupComponent {
  constructor() {
    interval(1000).pipe(
      takeUntilDestroyed() // Auto-unsubscribe on destroy
    ).subscribe(console.log);
  }
}

// Or inject DestroyRef for use outside constructor
@Component({...})
export class ManualCleanupComponent {
  private destroyRef = inject(DestroyRef);

  startTimer() {
    interval(1000).pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(console.log);
  }
}
```

## Testing Observables

```typescript
import { TestScheduler } from 'rxjs/testing';
import { map } from 'rxjs/operators';

describe('Operators', () => {
  let scheduler: TestScheduler;

  beforeEach(() => {
    scheduler = new TestScheduler((actual, expected) => {
      expect(actual).toEqual(expected);
    });
  });

  it('should double values', () => {
    scheduler.run(({ cold, expectObservable }) => {
      const source$ = cold('a-b-c|', { a: 1, b: 2, c: 3 });
      const expected =     'a-b-c|';
      const expectedValues = { a: 2, b: 4, c: 6 };

      const result$ = source$.pipe(map(x => x * 2));

      expectObservable(result$).toBe(expected, expectedValues);
    });
  });
});
```

## Summary

| Category | Operators |
|----------|-----------|
| Creation | `of`, `from`, `interval`, `timer`, `fromEvent` |
| Transform | `map`, `scan`, `reduce`, `pluck` |
| Filter | `filter`, `take`, `first`, `distinctUntilChanged` |
| Combine | `combineLatest`, `merge`, `concat`, `forkJoin`, `zip` |
| Flatten | `switchMap`, `mergeMap`, `concatMap`, `exhaustMap` |
| Error | `catchError`, `retry`, `throwError` |
| Timing | `debounceTime`, `throttleTime`, `delay`, `timeout` |
| Utility | `tap`, `finalize`, `share`, `shareReplay` |

## Next Steps

- [State Management](./14-state-management.md) - State management patterns
- [NgRx Store](./15-ngrx.md) - Redux-style state management

---

[Previous: Angular Signals](./12-signals.md) | [Back to Index](./README.md) | [Next: State Management](./14-state-management.md)
