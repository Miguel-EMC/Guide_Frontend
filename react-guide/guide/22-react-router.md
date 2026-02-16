# 22 - React Router

React Router provides client side routing, data loading, and mutations for SPAs.

## Install

```bash
npm i react-router
```

## Router Setup

```tsx
import { createBrowserRouter } from 'react-router';
import { RouterProvider } from 'react-router/dom';

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { index: true, element: <Home /> },
      { path: 'products/:id', element: <Product /> }
    ]
  }
]);

export function App() {
  return <RouterProvider router={router} />;
}
```

## Loaders and Actions

```tsx
import { Form, useLoaderData } from 'react-router';

type Product = { id: string; title: string };

export async function loader({ params }: { params: { id: string } }) {
  const res = await fetch(`/api/products/${params.id}`);
  if (!res.ok) throw new Response('Not found', { status: 404 });
  return res.json();
}

export async function action({ request }: { request: Request }) {
  const formData = await request.formData();
  await fetch('/api/cart', { method: 'POST', body: formData });
  return null;
}

export function ProductRoute() {
  const product = useLoaderData() as Product;
  return (
    <div>
      <h1>{product.title}</h1>
      <Form method="post">
        <button type="submit">Add to cart</button>
      </Form>
    </div>
  );
}
```

## Next Steps

- [Basic Data Fetching](./23-data-fetching.md)
- [Advanced Data Fetching](./24-advanced-data-fetching.md)

---

[Previous: UI Component Libraries](./21-ui-libraries.md) | [Back to Index](./README.md) | [Next: Basic Data Fetching](./23-data-fetching.md)
