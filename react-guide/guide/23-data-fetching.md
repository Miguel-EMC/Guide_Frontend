# 23 - Basic Data Fetching (Fetch, Axios)

Fetch data with hooks and handle loading, error, and cancellation states.

## Fetch with useEffect

```tsx
import { useEffect, useState } from 'react';

type User = { id: string; name: string };

export function Users() {
  const [data, setData] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function load() {
      try {
        const res = await fetch('/api/users', { signal: controller.signal });
        if (!res.ok) throw new Error('Failed to fetch');
        const json = (await res.json()) as User[];
        setData(json);
      } catch (err) {
        if (!controller.signal.aborted) setError('Request failed');
      } finally {
        if (!controller.signal.aborted) setLoading(false);
      }
    }

    load();
    return () => controller.abort();
  }, []);

  if (loading) return <p>Loading...</p>;
  if (error) return <p>{error}</p>;

  return (
    <ul>
      {data.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

## Tips

- Use AbortController for request cancellation
- Keep network logic in a small data layer
- Use React Router loaders or TanStack Query for caching

## Next Steps

- [Advanced Data Fetching](./24-advanced-data-fetching.md)
- [Code Splitting](./25-code-splitting.md)

---

[Previous: React Router](./22-react-router.md) | [Back to Index](./README.md) | [Next: Advanced Data Fetching](./24-advanced-data-fetching.md)
