# 14 - TypeScript with React

TypeScript makes components safer and easier to refactor by adding static types.

## Typing Props and State

```tsx
type User = { id: string; name: string };

type UserRowProps = {
  user: User;
  onSelect: (id: string) => void;
};

export function UserRow({ user, onSelect }: UserRowProps) {
  return (
    <button onClick={() => onSelect(user.id)}>
      {user.name}
    </button>
  );
}
```

```tsx
import { useState } from 'react';

type Status = 'idle' | 'loading' | 'error';

export function StatusChip() {
  const [status, setStatus] = useState<Status>('idle');
  return <span>{status}</span>;
}
```

## Typing Events

```tsx
function onSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
}

function onChange(e: React.ChangeEvent<HTMLInputElement>) {
  console.log(e.target.value);
}
```

## Reusable Types

```tsx
import type { ComponentProps, PropsWithChildren } from 'react';

type ButtonProps = ComponentProps<'button'> & { variant?: 'primary' | 'ghost' };

export function Button({ variant = 'primary', ...props }: ButtonProps) {
  return <button data-variant={variant} {...props} />;
}

export function Card({ children }: PropsWithChildren) {
  return <section className="card">{children}</section>;
}
```

## Tips

- Prefer type aliases for unions and object shapes
- Keep types close to the components that use them
- Enable strict mode in tsconfig for best safety

## Next Steps

- [State Management](./15-state-management.md)
- [Zustand](./16-zustand.md)

---

[Previous: Concurrent React](./13-concurrent-react.md) | [Back to Index](./README.md) | [Next: State Management](./15-state-management.md)
