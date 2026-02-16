# 28 - End-to-End Testing

E2E tests validate full user flows in the browser.

## Install Playwright

```bash
npm init playwright@latest
```

## Example Test

```ts
import { test, expect } from '@playwright/test';

test('user can log in', async ({ page }) => {
  await page.goto('http://localhost:5173/login');
  await page.getByLabel('Email').fill('user@example.com');
  await page.getByLabel('Password').fill('secret');
  await page.getByRole('button', { name: 'Sign in' }).click();
  await expect(page.getByText('Welcome back')).toBeVisible();
});
```

## Tips

- Run E2E tests in CI with a production build
- Use stable selectors and roles
- Keep E2E tests focused on critical flows

## Next Steps

- [Build Tools](./29-build-tools.md)
- [Deployment Strategies](./30-deployment.md)

---

[Previous: Unit Testing with RTL/Vitest](./27-testing.md) | [Back to Index](./README.md) | [Next: Build Tools](./29-build-tools.md)
