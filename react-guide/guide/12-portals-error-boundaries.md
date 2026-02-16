# 12 - Portals & Error Boundaries

Portals render children into a different part of the DOM. Error boundaries catch render errors and show a fallback UI.

## Portals for Modals

Add a portal root in index.html:

```html
<div id="root"></div>
<div id="modal-root"></div>
```

```tsx
import { createPortal } from 'react-dom';

type ModalProps = {
  open: boolean;
  onClose: () => void;
  children: React.ReactNode;
};

export function Modal({ open, onClose, children }: ModalProps) {
  if (!open) return null;
  const host = document.getElementById('modal-root');
  if (!host) return null;

  return createPortal(
    <div className="backdrop" onClick={onClose}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>,
    host
  );
}
```

## Error Boundaries

```tsx
import React from 'react';

type Props = { fallback: React.ReactNode; children: React.ReactNode };
type State = { hasError: boolean };

export class ErrorBoundary extends React.Component<Props, State> {
  state: State = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: unknown) {
    console.error('ErrorBoundary caught', error);
  }

  render() {
    if (this.state.hasError) return this.props.fallback;
    return this.props.children;
  }
}
```

```tsx
<ErrorBoundary fallback={<p>Something went wrong.</p>}>
  <Profile />
</ErrorBoundary>
```

## Notes

- Error boundaries do not catch errors in event handlers
- Wrap large sections or routes to contain crashes

## Next Steps

- [Concurrent React](./13-concurrent-react.md)
- [TypeScript with React](./14-typescript.md)

---

[Previous: Context API in Depth](./11-context-api.md) | [Back to Index](./README.md) | [Next: Concurrent React](./13-concurrent-react.md)
