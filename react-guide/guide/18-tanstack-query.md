# 18 - TanStack Query

TanStack Query handles server state: caching, deduping, retries, and background refetching.

## Install

```bash
npm i @tanstack/react-query
```

## Query Client Setup

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

export function AppProviders({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

## Fetch Data with useQuery

```tsx
import { useQuery } from '@tanstack/react-query';

type Todo = { id: string; title: string };

async function fetchTodos(): Promise<Todo[]> {
  const res = await fetch('/api/todos');
  if (!res.ok) throw new Error('Failed to load');
  return res.json();
}

export function Todos() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    staleTime: 60_000
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Something went wrong</p>;

  return (
    <ul>
      {data?.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

## Mutations and Invalidation

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';

async function createTodo(title: string) {
  const res = await fetch('/api/todos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title })
  });
  if (!res.ok) throw new Error('Failed to create');
  return res.json();
}

export function NewTodo() {
  const queryClient = useQueryClient();
  const mutation = useMutation({
    mutationFn: createTodo,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['todos'] })
  });

  return (
    <button onClick={() => mutation.mutate('Learn React 19')}>
      Add Todo
    </button>
  );
}
```

## Next Steps

- [CSS Modules & Styled Components](./19-styling.md)
- [Tailwind CSS Integration](./20-tailwind.md)

---

[Previous: Jotai & Recoil](./17-jotai-recoil.md) | [Back to Index](./README.md) | [Next: CSS Modules & Styled Components](./19-styling.md)
