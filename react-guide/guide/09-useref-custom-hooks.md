# 09 - useRef & Custom Hooks

`useRef` stores mutable values and references to DOM nodes. Custom hooks package reusable logic.

## useRef for DOM

```tsx
import { useRef } from 'react';

function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </div>
  );
}
```

## useRef for Mutable Values

```tsx
const renderCount = useRef(0);
renderCount.current += 1;
```

## Custom Hooks

```tsx
import { useEffect, useState } from 'react';

function useLocalStorage(key: string, initial: string) {
  const [value, setValue] = useState(() => {
    if (typeof window === 'undefined') return initial;
    return localStorage.getItem(key) ?? initial;
  });

  useEffect(() => {
    if (typeof window === 'undefined') return;
    localStorage.setItem(key, value);
  }, [key, value]);

  return [value, setValue] as const;
}
```

## Next Steps

- [Optimizations](./10-optimizations.md)
- [Context API in Depth](./11-context-api.md)

---

[Previous: useContext & useReducer](./08-usecontext-usereducer.md) | [Back to Index](./README.md) | [Next: Optimizations](./10-optimizations.md)
