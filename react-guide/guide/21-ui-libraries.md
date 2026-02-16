# 21 - UI Component Libraries

UI libraries help you ship faster while keeping design consistent.

## Common Choices

- Headless: Radix UI, Headless UI
- Component suites: MUI, Chakra UI, Ant Design
- Design systems: shadcn/ui, custom internal libraries

## Example (MUI)

```tsx
import Button from '@mui/material/Button';

export function SaveButton() {
  return <Button variant="contained">Save</Button>;
}
```

## Selection Criteria

- Accessibility and keyboard support
- Customization and theming
- Bundle size and tree shaking
- Community support and updates

## Next Steps

- [React Router](./22-react-router.md)
- [Basic Data Fetching](./23-data-fetching.md)

---

[Previous: Tailwind CSS Integration](./20-tailwind.md) | [Back to Index](./README.md) | [Next: React Router](./22-react-router.md)
