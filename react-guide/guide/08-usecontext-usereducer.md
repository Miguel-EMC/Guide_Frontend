# 08 - useContext & useReducer

Use Context to share data across component trees. Use Reducer for complex state transitions.

## useContext

```tsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext<'light' | 'dark'>('light');

function ThemeButton() {
  const theme = useContext(ThemeContext);
  return <button className={theme}>Theme</button>;
}
```

## useReducer

```tsx
import { useReducer } from 'react';

type State = { count: number };

type Action = { type: 'inc' } | { type: 'dec' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'inc':
      return { count: state.count + 1 };
    case 'dec':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

export function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'inc' })}>+</button>
    </div>
  );
}
```

## Context + Reducer

```tsx
import { createContext, useReducer } from 'react';
import type { Dispatch, ReactNode } from 'react';

const CounterContext = createContext<{ state: State; dispatch: Dispatch<Action> } | null>(null);

function CounterProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(reducer, { count: 0 });
  return (
    <CounterContext.Provider value={{ state, dispatch }}>
      {children}
    </CounterContext.Provider>
  );
}
```

## Next Steps

- [useRef & Custom Hooks](./09-useref-custom-hooks.md)
- [Optimizations](./10-optimizations.md)

---

[Previous: useState & useEffect](./07-usestate-useeffect.md) | [Back to Index](./README.md) | [Next: useRef & Custom Hooks](./09-useref-custom-hooks.md)
