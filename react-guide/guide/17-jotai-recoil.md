# 17 - Jotai & Recoil

Both Jotai and Recoil model state as atoms. Jotai is minimal and flexible. Recoil provides selectors and a larger API surface.

## Jotai Basics

```tsx
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);

export function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubled] = useAtom(doubledAtom);

  return (
    <div>
      <button onClick={() => setCount((c) => c + 1)}>Count: {count}</button>
      <p>Doubled: {doubled}</p>
    </div>
  );
}
```

## Recoil Basics

```tsx
import { atom, selector, useRecoilState, useRecoilValue, RecoilRoot } from 'recoil';

const countState = atom({ key: 'count', default: 0 });

const doubledSelector = selector({
  key: 'doubled',
  get: ({ get }) => get(countState) * 2
});

function Counter() {
  const [count, setCount] = useRecoilState(countState);
  const doubled = useRecoilValue(doubledSelector);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <p>Doubled: {doubled}</p>
    </div>
  );
}

export function App() {
  return (
    <RecoilRoot>
      <Counter />
    </RecoilRoot>
  );
}
```

## Tips

- Prefer Jotai for small to medium global state
- Use selectors for derived state instead of duplicating data
- Isolate atoms by domain to keep apps maintainable

## Next Steps

- [TanStack Query](./18-tanstack-query.md)
- [CSS Modules & Styled Components](./19-styling.md)

---

[Previous: Zustand](./16-zustand.md) | [Back to Index](./README.md) | [Next: TanStack Query](./18-tanstack-query.md)
