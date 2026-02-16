# 15 - Introduction to State Management

Not all state is equal. Choose the simplest tool that solves the problem.

## Types of State

- Local UI state (inputs, toggles, tabs)
- Shared UI state (theme, auth, layout)
- Server state (data from APIs)
- URL state (filters, pagination)

## Principles

- Keep state close to where it is used
- Prefer derived state over duplicated state
- Use reducers when state transitions are complex

## useReducer Example

```tsx
import { useReducer } from 'react';

type State = { query: string; page: number };
type Action =
  | { type: 'setQuery'; value: string }
  | { type: 'setPage'; value: number };

function reducer(state: State, action: Action): State {
  if (action.type === 'setQuery') return { ...state, query: action.value, page: 1 };
  if (action.type === 'setPage') return { ...state, page: action.value };
  return state;
}

export function SearchState() {
  const [state, dispatch] = useReducer(reducer, { query: '', page: 1 });

  return (
    <div>
      <input
        value={state.query}
        onChange={(e) => dispatch({ type: 'setQuery', value: e.target.value })}
      />
      <button onClick={() => dispatch({ type: 'setPage', value: state.page + 1 })}>
        Next Page
      </button>
    </div>
  );
}
```

## When to Adopt a Store

- Multiple distant components need the same data
- You need persistence, undo, or time travel
- You want explicit state transitions and devtools

## Next Steps

- [Zustand](./16-zustand.md)
- [Jotai & Recoil](./17-jotai-recoil.md)

---

[Previous: TypeScript with React](./14-typescript.md) | [Back to Index](./README.md) | [Next: Zustand](./16-zustand.md)
