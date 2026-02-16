# Reactive Forms

Reactive forms provide a model-driven approach to handling form inputs. They offer more control, are easier to test, and scale better for complex forms.

## Setup

Import ReactiveFormsModule:

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormControl, FormGroup, Validators } from '@angular/forms';

@Component({
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `...`
})
export class FormComponent {}
```

## FormControl

### Basic FormControl

```typescript
import { Component } from '@angular/core';
import { ReactiveFormsModule, FormControl } from '@angular/forms';

@Component({
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <input [formControl]="name" />
    <p>Value: {{ name.value }}</p>
    <p>Valid: {{ name.valid }}</p>
  `
})
export class BasicFormComponent {
  name = new FormControl('');
}
```

### FormControl with Validators

```typescript
import { Validators, FormControl } from '@angular/forms';

name = new FormControl('', [
  Validators.required,
  Validators.minLength(2),
  Validators.maxLength(50)
]);

email = new FormControl('', [
  Validators.required,
  Validators.email
]);

age = new FormControl(null, [
  Validators.required,
  Validators.min(18),
  Validators.max(120)
]);

website = new FormControl('', [
  Validators.pattern(/^https?:\/\/.+/)
]);
```

### Typed FormControls

```typescript
import { FormControl } from '@angular/forms';
// Strongly typed
const name = new FormControl<string>('John');
const age = new FormControl<number | null>(null);
const active = new FormControl<boolean>(true);

// Non-nullable
const required = new FormControl('', { nonNullable: true });
```

## FormGroup

### Basic FormGroup

```typescript
import { Component } from '@angular/core';
import { Validators, FormGroup, FormControl, ReactiveFormsModule } from '@angular/forms';
@Component({
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <div>
        <label for="name">Name:</label>
        <input id="name" formControlName="name" />
        @if (userForm.get('name')?.errors?.['required'] && userForm.get('name')?.touched) {
          <span class="error">Name is required</span>
        }
      </div>

      <div>
        <label for="email">Email:</label>
        <input id="email" formControlName="email" type="email" />
      </div>

      <button type="submit" [disabled]="userForm.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  userForm = new FormGroup({
    name: new FormControl('', [Validators.required]),
    email: new FormControl('', [Validators.required, Validators.email])
  });

  onSubmit() {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

### Nested FormGroups

```typescript
import { Validators, FormGroup, FormControl } from '@angular/forms';
userForm = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', [Validators.required, Validators.email]),
  address: new FormGroup({
    street: new FormControl(''),
    city: new FormControl(''),
    zipCode: new FormControl('', Validators.pattern(/^\d{5}$/))
  })
});
```

**Template:**

```html
<form [formGroup]="userForm">
  <input formControlName="name" />
  <input formControlName="email" />

  <div formGroupName="address">
    <input formControlName="street" placeholder="Street" />
    <input formControlName="city" placeholder="City" />
    <input formControlName="zipCode" placeholder="Zip Code" />
  </div>
</form>
```

### Typed FormGroup

```typescript
import { FormGroup, FormControl } from '@angular/forms';
interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
  age: FormControl<number | null>;
}

userForm = new FormGroup<UserForm>({
  name: new FormControl('', { nonNullable: true }),
  email: new FormControl('', { nonNullable: true }),
  age: new FormControl<number | null>(null)
});

// Access typed values
const name: string = this.userForm.controls.name.value;
```

## FormBuilder

Simplify form creation:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';

@Component({...})
export class FormBuilderComponent {
  private fb = inject(FormBuilder);

  userForm = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
    address: this.fb.group({
      street: [''],
      city: [''],
      zipCode: ['', Validators.pattern(/^\d{5}$/)]
    }),
    preferences: this.fb.group({
      newsletter: [false],
      notifications: [true]
    })
  });
}
```

### NonNullable FormBuilder

```typescript
import { inject } from '@angular/core';
import { NonNullableFormBuilder, Validators } from '@angular/forms';
private fb = inject(NonNullableFormBuilder);

// All controls are non-nullable by default
form = this.fb.group({
  name: ['', Validators.required],
  email: ['', Validators.email]
});

// Reset returns to initial values, not null
this.form.reset(); // { name: '', email: '' }
```

## FormArray

Dynamic list of controls:

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, Validators, FormArray } from '@angular/forms';
@Component({
  template: `
    <form [formGroup]="form">
      <div formArrayName="phones">
        @for (phone of phones.controls; track $index; let i = $index) {
          <div class="phone-row">
            <input [formControlName]="i" placeholder="Phone number" />
            <button type="button" (click)="removePhone(i)">Remove</button>
          </div>
        }
      </div>
      <button type="button" (click)="addPhone()">Add Phone</button>
    </form>
  `
})
export class PhoneListComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    phones: this.fb.array([
      this.fb.control('', Validators.required)
    ])
  });

  get phones() {
    return this.form.get('phones') as FormArray;
  }

  addPhone() {
    this.phones.push(this.fb.control('', Validators.required));
  }

  removePhone(index: number) {
    this.phones.removeAt(index);
  }
}
```

### FormArray with FormGroups

```typescript
import { Validators, FormGroup, FormArray } from '@angular/forms';
form = this.fb.group({
  users: this.fb.array<FormGroup>([])
});

get users() {
  return this.form.get('users') as FormArray;
}

addUser() {
  const userGroup = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]],
    role: ['user']
  });
  this.users.push(userGroup);
}

removeUser(index: number) {
  this.users.removeAt(index);
}
```

**Template:**

```html
<div formArrayName="users">
  @for (user of users.controls; track $index; let i = $index) {
    <div [formGroupName]="i" class="user-row">
      <input formControlName="name" placeholder="Name" />
      <input formControlName="email" placeholder="Email" />
      <select formControlName="role">
        <option value="user">User</option>
        <option value="admin">Admin</option>
      </select>
      <button (click)="removeUser(i)">Remove</button>
    </div>
  }
</div>
```

## Validation

### Built-in Validators

| Validator | Description |
|-----------|-------------|
| `Validators.required` | Field must have value |
| `Validators.requiredTrue` | Must be true (checkboxes) |
| `Validators.email` | Valid email format |
| `Validators.min(n)` | Minimum number value |
| `Validators.max(n)` | Maximum number value |
| `Validators.minLength(n)` | Minimum string length |
| `Validators.maxLength(n)` | Maximum string length |
| `Validators.pattern(regex)` | Match regex pattern |

### Custom Validators

```typescript
import { AbstractControl, ValidationErrors, ValidatorFn, Validators, FormControl } from '@angular/forms';

// Simple validator function
export function forbiddenNameValidator(forbiddenName: RegExp): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const forbidden = forbiddenName.test(control.value);
    return forbidden ? { forbiddenName: { value: control.value } } : null;
  };
}

// Usage
name = new FormControl('', [
  Validators.required,
  forbiddenNameValidator(/admin/i)
]);
```

### Cross-Field Validators

```typescript
import { Validators } from '@angular/forms';
export function passwordMatchValidator(): ValidatorFn {
  return (group: AbstractControl): ValidationErrors | null => {
    const password = group.get('password')?.value;
    const confirmPassword = group.get('confirmPassword')?.value;

    if (password !== confirmPassword) {
      return { passwordMismatch: true };
    }
    return null;
  };
}

// Usage
form = this.fb.group({
  password: ['', [Validators.required, Validators.minLength(8)]],
  confirmPassword: ['', Validators.required]
}, {
  validators: [passwordMatchValidator()]
});
```

### Async Validators

```typescript
import { AsyncValidatorFn, Validators, FormControl } from '@angular/forms';
import { catchError, map } from 'rxjs/operators';
import { Observable, of } from 'rxjs';

export function uniqueEmailValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkEmailExists(control.value).pipe(
      map(exists => exists ? { emailTaken: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage
email = new FormControl('', {
  validators: [Validators.required, Validators.email],
  asyncValidators: [uniqueEmailValidator(this.userService)],
  updateOn: 'blur'  // Run async validator on blur
});
```

### Displaying Errors

```html
<div class="form-field">
  <label for="email">Email</label>
  <input id="email" formControlName="email" />

  @if (email.errors && email.touched) {
    <div class="errors">
      @if (email.errors['required']) {
        <span>Email is required</span>
      }
      @if (email.errors['email']) {
        <span>Invalid email format</span>
      }
      @if (email.errors['emailTaken']) {
        <span>Email is already registered</span>
      }
    </div>
  }

  @if (email.pending) {
    <span class="loading">Checking availability...</span>
  }
</div>
```

### Error Helper

```typescript
import { Component, computed, input } from '@angular/core';
@Component({
  template: `
    <input formControlName="name" />
    <app-form-errors [control]="form.get('name')" [messages]="nameErrors" />
  `
})
export class FormComponent {
  nameErrors = {
    required: 'Name is required',
    minlength: 'Name must be at least 2 characters',
    forbiddenName: 'This name is not allowed'
  };
}

// Error component
@Component({
  selector: 'app-form-errors',
  standalone: true,
  template: `
    @if (control?.errors && (control?.dirty || control?.touched)) {
      <div class="errors">
        @for (error of errorMessages; track error) {
          <span class="error">{{ error }}</span>
        }
      </div>
    }
  `
})
export class FormErrorsComponent {
  control = input<AbstractControl | null>();
  messages = input<Record<string, string>>({});

  errorMessages = computed(() => {
    const ctrl = this.control();
    const msgs = this.messages();
    if (!ctrl?.errors) return [];

    return Object.keys(ctrl.errors)
      .filter(key => msgs[key])
      .map(key => msgs[key]);
  });
}
```

## Form State

### Control Properties

| Property | Description |
|----------|-------------|
| `value` | Current value |
| `valid` | All validators pass |
| `invalid` | Any validator fails |
| `pending` | Async validators running |
| `pristine` | Value not changed |
| `dirty` | Value has been changed |
| `touched` | Control has been blurred |
| `untouched` | Control never blurred |
| `errors` | Validation errors object |

### Form Methods

```typescript
// Set value
this.form.setValue({
  name: 'John',
  email: 'john@example.com'
});

// Patch value (partial)
this.form.patchValue({
  name: 'John'
});

// Reset form
this.form.reset();
this.form.reset({ name: 'Default Name' });

// Disable/Enable
this.form.disable();
this.form.enable();
this.form.get('email')?.disable();

// Mark as touched (for validation display)
this.form.markAllAsTouched();

// Get raw value (includes disabled)
const data = this.form.getRawValue();
```

## Dynamic Forms

```typescript
import { Component, OnInit } from '@angular/core';
import { Validators, FormGroup, FormControl } from '@angular/forms';
interface FormField {
  name: string;
  type: 'text' | 'email' | 'select' | 'checkbox';
  label: string;
  validators?: ValidatorFn[];
  options?: { value: string; label: string }[];
}

@Component({
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      @for (field of fields; track field.name) {
        <div class="field">
          <label [for]="field.name">{{ field.label }}</label>

          @switch (field.type) {
            @case ('text') {
              <input [id]="field.name" [formControlName]="field.name" />
            }
            @case ('email') {
              <input [id]="field.name" type="email" [formControlName]="field.name" />
            }
            @case ('select') {
              <select [id]="field.name" [formControlName]="field.name">
                @for (opt of field.options; track opt.value) {
                  <option [value]="opt.value">{{ opt.label }}</option>
                }
              </select>
            }
            @case ('checkbox') {
              <input [id]="field.name" type="checkbox" [formControlName]="field.name" />
            }
          }
        </div>
      }
      <button type="submit">Submit</button>
    </form>
  `
})
export class DynamicFormComponent implements OnInit {
  fields: FormField[] = [
    { name: 'name', type: 'text', label: 'Name', validators: [Validators.required] },
    { name: 'email', type: 'email', label: 'Email', validators: [Validators.required, Validators.email] },
    { name: 'country', type: 'select', label: 'Country', options: [
      { value: 'us', label: 'United States' },
      { value: 'uk', label: 'United Kingdom' }
    ]},
    { name: 'newsletter', type: 'checkbox', label: 'Subscribe to newsletter' }
  ];

  form!: FormGroup;

  ngOnInit() {
    this.form = this.buildForm();
  }

  buildForm(): FormGroup {
    const group: Record<string, FormControl> = {};

    this.fields.forEach(field => {
      group[field.name] = new FormControl(
        field.type === 'checkbox' ? false : '',
        field.validators || []
      );
    });

    return new FormGroup(group);
  }

  onSubmit() {
    console.log(this.form.value);
  }
}
```

## Complete Form Example

```typescript
import { Component, inject } from '@angular/core';
import { NonNullableFormBuilder, Validators, ReactiveFormsModule } from '@angular/forms';
interface UserFormValue {
  personalInfo: {
    firstName: string;
    lastName: string;
    email: string;
  };
  address: {
    street: string;
    city: string;
    zipCode: string;
  };
  preferences: {
    newsletter: boolean;
    theme: 'light' | 'dark';
  };
}

@Component({
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <section formGroupName="personalInfo">
        <h3>Personal Information</h3>
        <div class="field">
          <label>First Name</label>
          <input formControlName="firstName" />
        </div>
        <div class="field">
          <label>Last Name</label>
          <input formControlName="lastName" />
        </div>
        <div class="field">
          <label>Email</label>
          <input formControlName="email" type="email" />
        </div>
      </section>

      <section formGroupName="address">
        <h3>Address</h3>
        <div class="field">
          <label>Street</label>
          <input formControlName="street" />
        </div>
        <div class="field">
          <label>City</label>
          <input formControlName="city" />
        </div>
        <div class="field">
          <label>Zip Code</label>
          <input formControlName="zipCode" />
        </div>
      </section>

      <section formGroupName="preferences">
        <h3>Preferences</h3>
        <label>
          <input type="checkbox" formControlName="newsletter" />
          Subscribe to newsletter
        </label>
        <div>
          <label>
            <input type="radio" formControlName="theme" value="light" /> Light
          </label>
          <label>
            <input type="radio" formControlName="theme" value="dark" /> Dark
          </label>
        </div>
      </section>

      <button type="submit" [disabled]="form.invalid">
        {{ form.valid ? 'Submit' : 'Please fill all required fields' }}
      </button>
    </form>
  `
})
export class CompleteFormComponent {
  private fb = inject(NonNullableFormBuilder);

  form = this.fb.group({
    personalInfo: this.fb.group({
      firstName: ['', Validators.required],
      lastName: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]]
    }),
    address: this.fb.group({
      street: [''],
      city: [''],
      zipCode: ['', Validators.pattern(/^\d{5}$/)]
    }),
    preferences: this.fb.group({
      newsletter: [false],
      theme: ['light' as const]
    })
  });

  onSubmit() {
    if (this.form.valid) {
      const value: UserFormValue = this.form.getRawValue();
      console.log('Form submitted:', value);
    }
  }
}
```

## Summary

| Concept | Description |
|---------|-------------|
| `FormControl` | Single input control |
| `FormGroup` | Group of controls |
| `FormArray` | Dynamic list of controls |
| `FormBuilder` | Factory for creating forms |
| `Validators` | Built-in validation functions |
| Custom Validators | Create your own validation logic |
| Async Validators | Validation with async operations |

## Next Steps

- [Signal Forms (v21)](./10-signal-forms.md) - New signal-based forms
- [HTTP Client](./11-http-client.md) - Send form data to server

---

[Previous: Routing](./08-routing.md) | [Back to Index](./README.md) | [Next: Signal Forms](./10-signal-forms.md)
