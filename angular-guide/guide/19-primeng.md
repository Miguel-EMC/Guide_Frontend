# PrimeNG

PrimeNG is a rich UI component library for Angular. It includes ready-to-use widgets, data tables, charts, and theming options.

## Installation

```bash
npm install primeng primeicons @primeuix/themes
```

## Configure PrimeNG (Standalone)

```typescript
import { ApplicationConfig } from '@angular/core';
import { providePrimeNG } from 'primeng/config';
import Aura from '@primeuix/themes/aura';

export const appConfig: ApplicationConfig = {
  providers: [
    providePrimeNG({
      theme: {
        preset: Aura
      }
    })
  ]
};
```

## Example: Button and Table

```typescript
import { Component } from '@angular/core';
import { ButtonModule } from 'primeng/button';
import { TableModule } from 'primeng/table';

@Component({
  standalone: true,
  imports: [ButtonModule, TableModule],
  template: `
    <p-button label="Create"></p-button>

    <p-table [value]="users">
      <ng-template pTemplate="header">
        <tr>
          <th>Name</th>
          <th>Role</th>
        </tr>
      </ng-template>
      <ng-template pTemplate="body" let-user>
        <tr>
          <td>{{ user.name }}</td>
          <td>{{ user.role }}</td>
        </tr>
      </ng-template>
    </p-table>
  `
})
export class PrimeExampleComponent {
  users = [
    { name: 'Ada', role: 'Admin' },
    { name: 'Grace', role: 'Editor' }
  ];
}
```

## Best Practices

- Import only the modules you use
- Use PrimeNG themes as a base and layer your design system
- Favor data table virtualization for large datasets

## Next Steps

- [Tailwind CSS Integration](./20-tailwind.md) - Utility-first styling
- [Angular Material](./17-angular-material.md) - Alternative UI kit

---

[Previous: Libraries and Integrations](./18-libraries-and-integrations.md) | [Back to Index](./README.md) | [Next: Tailwind CSS Integration](./20-tailwind.md)
