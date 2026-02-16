# 11 - Context API in Depth

Context lets you share values across the component tree without prop drilling. Use it for app level dependencies, not for high frequency updates.

## When to Use Context

- Theme, locale, feature flags
- Auth session and user profile
- App services like analytics or API clients

## Basic Provider and Hook

```tsx
import { createContext, useContext, useMemo, useState } from 'react';

type Theme = 'light' | 'dark';
type ThemeContextValue = { theme: Theme; setTheme: (t: Theme) => void };

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
}
```

## Split Contexts to Reduce Re-renders

```tsx
const ThemeStateContext = createContext<Theme>('light');
const ThemeDispatchContext = createContext<(t: Theme) => void>(() => {});

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  return (
    <ThemeStateContext.Provider value={theme}>
      <ThemeDispatchContext.Provider value={setTheme}>
        {children}
      </ThemeDispatchContext.Provider>
    </ThemeStateContext.Provider>
  );
}
```

## Context + Reducer for Complex State

```tsx
type Action = { type: 'toggle' } | { type: 'set'; value: Theme };

function reducer(state: Theme, action: Action): Theme {
  if (action.type === 'toggle') return state === 'light' ? 'dark' : 'light';
  if (action.type === 'set') return action.value;
  return state;
}
```

## Tips

- Memoize provider values to avoid cascading renders
- Keep fast changing state out of context
- For large global state, consider a dedicated store

## Next Steps

- [Portals & Error Boundaries](./12-portals-error-boundaries.md)
- [Concurrent React](./13-concurrent-react.md)

---

[Previous: useMemo, useCallback & Optimizations](./10-optimizations.md) | [Back to Index](./README.md) | [Next: Portals & Error Boundaries](./12-portals-error-boundaries.md)
