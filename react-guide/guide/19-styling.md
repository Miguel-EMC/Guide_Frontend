# 19 - CSS Modules & Styled Components

React supports many styling approaches. Choose one that fits your team and scale.

## CSS Modules

```css
/* Card.module.css */
.card {
  padding: 16px;
  border-radius: 12px;
  background: white;
}
```

```tsx
import styles from './Card.module.css';

export function Card({ title }: { title: string }) {
  return (
    <section className={styles.card}>
      <h3>{title}</h3>
    </section>
  );
}
```

## Styled Components

```tsx
import styled from 'styled-components';

const Button = styled.button`
  padding: 10px 14px;
  border-radius: 8px;
  background: black;
  color: white;
`;

export function CTA() {
  return <Button>Subscribe</Button>;
}
```

## Tips

- Use CSS variables for themes and design tokens
- Keep global styles minimal
- Prefer co-located styles for components

## Next Steps

- [Tailwind CSS Integration](./20-tailwind.md)
- [UI Component Libraries](./21-ui-libraries.md)

---

[Previous: TanStack Query](./18-tanstack-query.md) | [Back to Index](./README.md) | [Next: Tailwind CSS Integration](./20-tailwind.md)
