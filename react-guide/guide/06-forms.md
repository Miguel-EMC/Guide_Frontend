# 06 - Forms

React supports controlled and uncontrolled forms. React 19 also introduces Actions for async form handling.

## Controlled Inputs

```tsx
import { useState } from 'react';

function LoginForm() {
  const [email, setEmail] = useState('');

  return (
    <form>
      <input
        name="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
    </form>
  );
}
```

## Uncontrolled Inputs

```tsx
import { useRef, type FormEvent } from 'react';

function SearchForm() {
  const ref = useRef<HTMLInputElement>(null);

  const submit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    console.log(ref.current?.value);
  };

  return (
    <form onSubmit={submit}>
      <input name="query" ref={ref} />
      <button type="submit">Search</button>
    </form>
  );
}
```

## React 19 Actions (Forms)

```tsx
import { useActionState } from 'react';
import { useFormStatus } from 'react-dom';

async function loginAction(prev: { error?: string }, formData: FormData) {
  const email = formData.get('email');
  if (!email) return { error: 'Email is required' };
  return { error: undefined };
}

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Submitting...' : 'Submit'}</button>;
}

export function LoginWithAction() {
  const [state, formAction] = useActionState(loginAction, { error: undefined });

  return (
    <form action={formAction}>
      <input name="email" type="email" />
      {state.error && <p className="error">{state.error}</p>}
      <SubmitButton />
    </form>
  );
}
```

## Optimistic UI

```tsx
import { startTransition, useOptimistic } from 'react';

function LikeButton({ likes, onSave }: { likes: number; onSave: (n: number) => Promise<void> }) {
  const [optimistic, setOptimistic] = useOptimistic(likes);

  return (
    <button
      onClick={() => {
        startTransition(async () => {
          const next = optimistic + 1;
          setOptimistic(next);
          await onSave(next);
        });
      }}
    >
      ❤️ {optimistic}
    </button>
  );
}
```

## Next Steps

- [useState & useEffect](./07-usestate-useeffect.md)
- [useContext & useReducer](./08-usecontext-usereducer.md)

---

[Previous: Handling Events](./05-events.md) | [Back to Index](./README.md) | [Next: useState & useEffect](./07-usestate-useeffect.md)
