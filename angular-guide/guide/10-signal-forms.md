# Signal Forms (v21 - Experimental)

Angular v21 introduces Signal Forms as an experimental feature that provides a signal-based approach to forms. The API is still evolving, so use it with caution in production.

## Overview

Signal Forms are designed to integrate directly with signals and reduce boilerplate compared to reactive forms. You define a model signal and use the `form()` builder to attach validation and metadata.

## Setup

```typescript
import { Component, signal } from '@angular/core';
import { FormField, form, Validators } from '@angular/forms/signals';
import { Validators } from '@angular/forms';

@Component({
  standalone: true,
  imports: [FormField],
  template: `...`
})
export class LoginFormComponent {}
```

### Basic Example

```typescript
import { Component, signal, computed, model } from '@angular/core';
import { FormField, form, Validators } from '@angular/forms/signals';
import { Validators } from '@angular/forms';

interface LoginModel {
  email: string;
  password: string;
}

@Component({
  standalone: true,
  imports: [FormField],
  template: `
    <form class="card">
      <label>
        Email
        <input type="email" [formField]="loginForm.email" />
      </label>

      @if (loginForm.email().errors()?.['email']) {
        <p class="error">Invalid email</p>
      }

      <label>
        Password
        <input type="password" [formField]="loginForm.password" />
      </label>

      <button type="button" [disabled]="!canSubmit()" (click)="submit()">
        Sign in
      </button>
    </form>
  `
})
export class LoginFormComponent {
  model = signal<LoginModel>({ email: '', password: '' });

  loginForm = form(this.model, (login) => {
    Validators.required(login.email);
    Validators.email(login.email);
    Validators.required(login.password);
  });

  canSubmit = computed(() =>
    !this.loginForm.email().errors() && !this.loginForm.password().errors()
  );

  submit() {
    if (!this.canSubmit()) return;
    console.log('Submit', this.model());
  }
}
```

## Field State and Errors

Each field exposes signal-based state like `touched()`, `dirty()`, `valid()`, and `errors()`:

```html
<input type="text" [formField]="profileForm.name" />
@if (profileForm.name().touched() && profileForm.name().errors()?.['required']) {
  <span class="error">Name is required</span>
}
```

## Nested Models

```typescript
import { signal } from '@angular/core';
import { Validators } from '@angular/forms';
interface ProfileModel {
  name: string;
  address: {
    city: string;
    country: string;
  };
}

model = signal<ProfileModel>({ name: '', address: { city: '', country: '' } });

profileForm = form(this.model, (profile) => {
  Validators.required(profile.name);
  Validators.required(profile.address.city);
});
```

```html
<input [formField]="profileForm.name" />
<input [formField]="profileForm.address.city" />
```

## Async Validation (Manual Pattern)

Because Signal Forms are experimental, async validation patterns may evolve. A safe approach is to drive async checks with signals and show errors explicitly.

```typescript
import { signal } from '@angular/core';
emailTaken = signal(false);

checkEmail = async (email: string) => {
  if (!email) return;
  const available = await this.api.isEmailAvailable(email);
  this.emailTaken.set(!available);
};
```

```html
@if (emailTaken()) {
  <p class="error">Email is already in use</p>
}
```

## When to Use Signal Forms

- You are starting new forms in v21 and want signal-based APIs
- You want a smaller, more direct API than reactive forms
- You are okay with experimental APIs and potential breaking changes

## Best Practices

- Keep validation logic inside the `form()` builder
- Use computed signals for derived UI state
- Avoid mixing signal forms with reactive forms in the same component
- Gate experimental usage behind a feature flag if shipping to production

## Next Steps

- [Reactive Forms](./09-reactive-forms.md) - Traditional form APIs
- [Signals](./12-signals.md) - Signal fundamentals

---

[Previous: Reactive Forms](./09-reactive-forms.md) | [Back to Index](./README.md) | [Next: HTTP Client](./11-http-client.md)
