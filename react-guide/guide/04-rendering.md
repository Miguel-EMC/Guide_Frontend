# 04 - Conditional & List Rendering

Render UI based on state and data structures.

## Conditional Rendering

```tsx
function Status({ isOnline }: { isOnline: boolean }) {
  return <p>{isOnline ? 'Online' : 'Offline'}</p>;
}
```

```tsx
function Profile({ user }: { user?: { name: string } }) {
  if (!user) return <p>Loading...</p>;
  return <h3>{user.name}</h3>;
}
```

## List Rendering

```tsx
function TodoList({ todos }: { todos: { id: string; title: string }[] }) {
  return (
    <ul>
      {todos.map((todo) => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
```

### Keys

- Use stable keys (`id`), not array indices
- Keys help React reconcile efficiently

## Empty States

```tsx
function Results({ items }: { items: string[] }) {
  if (items.length === 0) return <p>No results found.</p>;
  return (
    <ul>
      {items.map((item) => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

## Next Steps

- [Handling Events](./05-events.md)
- [Forms](./06-forms.md)

---

[Previous: Props & State](./03-props-state.md) | [Back to Index](./README.md) | [Next: Handling Events](./05-events.md)
