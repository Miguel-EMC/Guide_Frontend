# 24 - Advanced Data Fetching (SWR, React Query)

Advanced libraries give you caching, revalidation, retries, and stale data control.

## SWR Example

```tsx
import useSWR from 'swr';

type Repo = { id: number; name: string };

const fetcher = (url: string) => fetch(url).then((r) => r.json());

export function Repos() {
  const { data, error, isLoading } = useSWR<Repo[]>('/api/repos', fetcher);

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Failed to load</p>;

  return (
    <ul>
      {data?.map((repo) => (
        <li key={repo.id}>{repo.name}</li>
      ))}
    </ul>
  );
}
```

## When to Use Each

- SWR: lightweight, great for simple caching and revalidation
- TanStack Query: advanced caching, mutations, and devtools
- React Router loaders: route driven data and mutations

## Tips

- Normalize cache keys for consistent invalidation
- Use optimistic updates for fast UX
- Avoid mixing multiple server state libraries in one app

## Next Steps

- [Code Splitting](./25-code-splitting.md)
- [Performance Best Practices](./26-performance.md)

---

[Previous: Basic Data Fetching](./23-data-fetching.md) | [Back to Index](./README.md) | [Next: Code Splitting](./25-code-splitting.md)
