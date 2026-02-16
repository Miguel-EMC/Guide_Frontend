# 05 - Handling Events

React uses synthetic events for consistent behavior across browsers.

## Basic Events

```tsx
export function Clicker() {
  const handleClick = () => {
    console.log('Clicked');
  };

  return <button onClick={handleClick}>Click</button>;
}
```

## Event Object

```tsx
function InputDemo() {
  return (
    <input
      onChange={(e) => console.log(e.target.value)}
      placeholder="Type..."
    />
  );
}
```

## Prevent Default

```tsx
function Link() {
  return (
    <a href="/" onClick={(e) => e.preventDefault()}>
      Prevent navigation
    </a>
  );
}
```

## Keyboard Events

```tsx
<input
  onKeyDown={(e) => {
    if (e.key === 'Enter') console.log('Submit');
  }}
/>
```

## Next Steps

- [Forms](./06-forms.md)
- [useState & useEffect](./07-usestate-useeffect.md)

---

[Previous: Conditional & List Rendering](./04-rendering.md) | [Back to Index](./README.md) | [Next: Forms](./06-forms.md)
