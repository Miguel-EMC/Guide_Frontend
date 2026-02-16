# 07 - useState & useEffect

These are the most common hooks for local state and side effects.

## useState

```tsx
import { useState } from 'react';

function Toggle() {
  const [on, setOn] = useState(false);
  return <button onClick={() => setOn((v) => !v)}>{on ? 'On' : 'Off'}</button>;
}
```

## useEffect

```tsx
import { useEffect, useState } from 'react';

function UserProfile({ id }: { id: string }) {
  const [user, setUser] = useState<{ name: string } | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      const res = await fetch(`/api/users/${id}`, { signal: controller.signal });
      const data = (await res.json()) as { name: string };
      setUser(data);
    }

    load().catch(() => {
      if (!controller.signal.aborted) setUser(null);
    });

    return () => controller.abort();
  }, [id]);

  if (!user) return <p>Loading...</p>;
  return <h3>{user.name}</h3>;
}
```

### Effect Tips

- Keep dependency arrays accurate
- Use cleanup for subscriptions and timers
- Avoid putting derived values in state

## Next Steps

- [useContext & useReducer](./08-usecontext-usereducer.md)
- [useRef & Custom Hooks](./09-useref-custom-hooks.md)

---

[Previous: Forms](./06-forms.md) | [Back to Index](./README.md) | [Next: useContext & useReducer](./08-usecontext-usereducer.md)
