# 27 - Unit Testing with RTL and Vitest

Unit tests focus on component behavior and user interactions.

## Install

```bash
npm i -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

## Vitest Config

```ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: './src/test/setup.ts',
    globals: true
  }
});
```

## Setup File

```ts
// src/test/setup.ts
import '@testing-library/jest-dom';
```

## Example Test

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

it('increments when clicked', async () => {
  render(<Counter />);
  const user = userEvent.setup();
  await user.click(screen.getByRole('button', { name: /count/i }));
  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});
```

## Tips

- Test behavior, not implementation details
- Prefer userEvent over fireEvent for realism
- Mock network requests at the boundary

## Next Steps

- [End-to-End Testing](./28-e2e-testing.md)
- [Build Tools](./29-build-tools.md)

---

[Previous: Performance Best Practices](./26-performance.md) | [Back to Index](./README.md) | [Next: End-to-End Testing](./28-e2e-testing.md)
