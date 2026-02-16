# NgRx Store

NgRx Store is a Redux-style state management library for Angular. It provides a single immutable store, actions, reducers, selectors, and effects for predictable, testable state flow.

## When to Use NgRx

- Large applications with complex shared state
- Teams that need predictable state transitions
- Heavy async workflows and side effects
- Time-travel debugging with devtools

## Installation

```bash
npm install @ngrx/store @ngrx/effects @ngrx/store-devtools @ngrx/entity
```

## Core Concepts

- **Actions**: Events that describe what happened
- **Reducers**: Pure functions that calculate the next state
- **Selectors**: Read-only queries over state
- **Effects**: Side effects for async work
- **Entity**: Helpers for normalized collections

## Example: Todos Feature

### Actions

```typescript
import { createActionGroup, emptyProps, props } from '@ngrx/store';

export const TodosActions = createActionGroup({
  source: 'Todos',
  events: {
    'Load Todos': emptyProps(),
    'Load Todos Success': props<{ todos: Todo[] }>(),
    'Load Todos Failure': props<{ error: string }>(),
    'Add Todo': props<{ title: string }>(),
    'Toggle Todo': props<{ id: string }>()
  }
});
```

### Reducer + Entity

```typescript
import { createReducer, on } from '@ngrx/store';
import { createEntityAdapter, EntityState } from '@ngrx/entity';
import { TodosActions } from './todos.actions';

export interface Todo {
  id: string;
  title: string;
  completed: boolean;
}

export interface TodosState extends EntityState<Todo> {
  loaded: boolean;
  error: string | null;
}

export const adapter = createEntityAdapter<Todo>({
  selectId: (todo) => todo.id
});

const initialState: TodosState = adapter.getInitialState({
  loaded: false,
  error: null
});

export const todosReducer = createReducer(
  initialState,
  on(TodosActions.loadTodosSuccess, (state, { todos }) =>
    adapter.setAll(todos, { ...state, loaded: true, error: null })
  ),
  on(TodosActions.loadTodosFailure, (state, { error }) => ({
    ...state,
    error
  })),
  on(TodosActions.addTodo, (state, { title }) =>
    adapter.addOne({ id: crypto.randomUUID(), title, completed: false }, state)
  ),
  on(TodosActions.toggleTodo, (state, { id }) => {
    const todo = state.entities[id];
    if (!todo) return state;
    return adapter.updateOne({
      id,
      changes: { completed: !todo.completed }
    }, state);
  })
);
```

### Selectors

```typescript
import { createFeatureSelector, createSelector } from '@ngrx/store';
import { adapter, TodosState } from './todos.reducer';

export const selectTodosState = createFeatureSelector<TodosState>('todos');

const { selectAll } = adapter.getSelectors();

export const selectAllTodos = createSelector(
  selectTodosState,
  selectAll
);

export const selectTodosLoaded = createSelector(
  selectTodosState,
  (state) => state.loaded
);
```

### Effects

```typescript
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { catchError, map, of, switchMap } from 'rxjs';
import { TodosActions } from './todos.actions';
import { TodosApi } from '../data/todos.api';

@Injectable()
export class TodosEffects {
  loadTodos$ = createEffect(() =>
    this.actions$.pipe(
      ofType(TodosActions.loadTodos),
      switchMap(() =>
        this.api.getTodos().pipe(
          map((todos) => TodosActions.loadTodosSuccess({ todos })),
          catchError((error) =>
            of(TodosActions.loadTodosFailure({ error: String(error) }))
          )
        )
      )
    )
  );

  constructor(private actions$: Actions, private api: TodosApi) {}
}
```

### Register Feature in Standalone App

```typescript
import { ApplicationConfig, isDevMode } from '@angular/core';
import { provideStore, provideState } from '@ngrx/store';
import { provideEffects } from '@ngrx/effects';
import { provideStoreDevtools } from '@ngrx/store-devtools';
import { todosReducer } from './state/todos.reducer';
import { TodosEffects } from './state/todos.effects';

export const appConfig: ApplicationConfig = {
  providers: [
    provideStore(),
    provideState({ name: 'todos', reducer: todosReducer }),
    provideEffects([TodosEffects]),
    provideStoreDevtools({ maxAge: 25, logOnly: !isDevMode() })
  ]
};
```

### Component Usage

```typescript
import { Component, inject } from '@angular/core';
import { AsyncPipe } from '@angular/common';
import { Store } from '@ngrx/store';
import { TodosActions } from './state/todos.actions';
import { selectAllTodos } from './state/todos.selectors';

@Component({
  standalone: true,
  imports: [AsyncPipe],
  template: `
    <button (click)="load()">Load</button>
    <ul>
      @for (todo of todos$ | async; track todo.id) {
        <li>{{ todo.title }}</li>
      }
    </ul>
  `
})
export class TodosComponent {
  private store = inject(Store);
  todos$ = this.store.select(selectAllTodos);

  load() {
    this.store.dispatch(TodosActions.loadTodos());
  }
}
```

## Testing

```typescript
import { inject } from '@angular/core';
import { provideMockStore, MockStore } from '@ngrx/store/testing';

TestBed.configureTestingModule({
  providers: [provideMockStore({ initialState: { todos: { loaded: false } } })]
});

const store = TestBed.inject(MockStore);
```

## Best Practices

- Keep actions descriptive and event-based
- Normalize collections with Entity
- Use feature-level state slices
- Keep reducers pure and side-effect free
- Limit effects to orchestration (API, storage, navigation)

## Next Steps

- [Signal Store](./16-signal-store.md) - Signal-first store patterns
- [Testing](./22-testing.md) - Store testing strategies

---

[Previous: State Management](./14-state-management.md) | [Back to Index](./README.md) | [Next: Signal Store](./16-signal-store.md)
