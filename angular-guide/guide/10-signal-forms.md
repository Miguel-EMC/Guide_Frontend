# Signal Forms (v21)

Angular v21 introduces Signal Forms, a new streamlined, signal-based approach to forms. Signal Forms integrate seamlessly with Angular's reactivity model and provide a more intuitive API.

## Overview

Signal Forms replace traditional reactive forms with signal-based primitives:

| Reactive Forms | Signal Forms |
|----------------|--------------|
| `FormControl` | `formControl()` |
| `FormGroup` | `formGroup()` |
| `FormArray` | `formArray()` |
| Observable-based | Signal-based |

## Setup

```typescript
import { Component } from '@angular/core';
import { SignalFormsModule, formControl, formGroup, formArray } from '@angular/forms';

@Component({
  standalone: true,
  imports: [SignalFormsModule],
  template: `...`
})
export class FormComponent {}
```

## Basic Signal Form

### Simple Control

```typescript
import { Component, computed } from '@angular/core';
import { formControl, SignalFormsModule } from '@angular/forms';

@Component({
  standalone: true,
  imports: [SignalFormsModule],
  template: `
    <input [formControl]="name" />
    <p>Value: {{ name.value() }}</p>
    <p>Valid: {{ name.valid() }}</p>
    <p>Touched: {{ name.touched() }}</p>
  `
})
export class BasicComponent {
  name = formControl('', {
    validators: [Validators.required, Validators.minLength(2)]
  });

  // Computed based on form state
  canSubmit = computed(() =>
    this.name.valid() && !this.name.pristine()
  );
}
```

### Form Group

```typescript
import { formControl, formGroup, Validators } from '@angular/forms';

@Component({
  standalone: true,
  imports: [SignalFormsModule],
  template: `
    <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
      <input formControlName="name" placeholder="Name" />
      <input formControlName="email" type="email" placeholder="Email" />

      @if (userForm.controls.email.errors()?.['email']) {
        <span class="error">Invalid email format</span>
      }

      <button [disabled]="!userForm.valid()">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  userForm = formGroup({
    name: formControl('', {
      validators: [Validators.required]
    }),
    email: formControl('', {
      validators: [Validators.required, Validators.email]
    })
  });

  onSubmit() {
    if (this.userForm.valid()) {
      console.log(this.userForm.value());
    }
  }
}
```

## Typed Signal Forms

Signal Forms are fully typed by default:

```typescript
interface UserFormModel {
  name: string;
  email: string;
  age: number | null;
}

@Component({...})
export class TypedFormComponent {
  form = formGroup<UserFormModel>({
    name: formControl(''),
    email: formControl(''),
    age: formControl<number | null>(null)
  });

  // Value is typed
  submit() {
    const value: UserFormModel = this.form.value();
    console.log(value.name); // string
    console.log(value.age);  // number | null
  }
}
```

## Signal Form State

All form state is exposed as signals:

```typescript
@Component({
  template: `
    <input [formControl]="email" />

    <div class="status">
      <span [class.valid]="email.valid()">
        {{ email.valid() ? '✓ Valid' : '✗ Invalid' }}
      </span>
      <span [class.touched]="email.touched()">
        {{ email.touched() ? 'Touched' : 'Untouched' }}
      </span>
      <span [class.dirty]="email.dirty()">
        {{ email.dirty() ? 'Modified' : 'Pristine' }}
      </span>
    </div>

    @if (email.pending()) {
      <span class="loading">Validating...</span>
    }

    @if (email.errors(); as errors) {
      <div class="errors">
        @for (error of Object.keys(errors); track error) {
          <p>{{ error }}: {{ errors[error] | json }}</p>
        }
      </div>
    }
  `
})
export class FormStateComponent {
  email = formControl('', {
    validators: [Validators.required, Validators.email],
    asyncValidators: [this.uniqueEmailValidator()]
  });
}
```

### State Signals

| Signal | Description |
|--------|-------------|
| `value()` | Current value |
| `valid()` | All validators pass |
| `invalid()` | Any validator fails |
| `pending()` | Async validators running |
| `pristine()` | Value unchanged |
| `dirty()` | Value has changed |
| `touched()` | Has been blurred |
| `untouched()` | Never blurred |
| `errors()` | Validation errors |
| `status()` | 'VALID', 'INVALID', 'PENDING' |

## Nested Form Groups

```typescript
@Component({
  template: `
    <form [formGroup]="form">
      <input formControlName="name" placeholder="Name" />

      <div formGroupName="address">
        <input formControlName="street" placeholder="Street" />
        <input formControlName="city" placeholder="City" />
        <input formControlName="zipCode" placeholder="Zip" />
      </div>

      <div formGroupName="contact">
        <input formControlName="phone" placeholder="Phone" />
        <input formControlName="email" placeholder="Email" />
      </div>
    </form>
  `
})
export class NestedFormComponent {
  form = formGroup({
    name: formControl('', Validators.required),
    address: formGroup({
      street: formControl(''),
      city: formControl(''),
      zipCode: formControl('', Validators.pattern(/^\d{5}$/))
    }),
    contact: formGroup({
      phone: formControl(''),
      email: formControl('', Validators.email)
    })
  });

  // Access nested values
  cityValue = computed(() => this.form.controls.address.controls.city.value());

  // Check nested validity
  addressValid = computed(() => this.form.controls.address.valid());
}
```

## Signal Form Array

```typescript
@Component({
  template: `
    <div [formGroup]="form">
      <div formArrayName="items">
        @for (item of items().controls; track $index; let i = $index) {
          <div class="item-row">
            <input [formControlName]="i" placeholder="Item {{ i + 1 }}" />
            <button type="button" (click)="removeItem(i)">Remove</button>
          </div>
        }
      </div>

      <button type="button" (click)="addItem()">Add Item</button>

      <p>Total items: {{ items().length }}</p>
      <p>All valid: {{ items().valid() }}</p>
    </div>
  `
})
export class FormArrayComponent {
  form = formGroup({
    items: formArray([
      formControl('', Validators.required)
    ])
  });

  items = computed(() => this.form.controls.items);

  addItem() {
    this.form.controls.items.push(
      formControl('', Validators.required)
    );
  }

  removeItem(index: number) {
    this.form.controls.items.removeAt(index);
  }
}
```

### FormArray with Groups

```typescript
interface ItemForm {
  name: string;
  quantity: number;
  price: number;
}

@Component({
  template: `
    <div formArrayName="items">
      @for (item of itemsArray.controls; track $index; let i = $index) {
        <div [formGroupName]="i" class="item-form">
          <input formControlName="name" placeholder="Name" />
          <input formControlName="quantity" type="number" placeholder="Qty" />
          <input formControlName="price" type="number" placeholder="Price" />
          <span>Subtotal: {{ getSubtotal(i)() | currency }}</span>
          <button (click)="removeItem(i)">×</button>
        </div>
      }
    </div>
    <p>Total: {{ total() | currency }}</p>
  `
})
export class ItemListComponent {
  form = formGroup({
    items: formArray<FormGroup<ItemForm>>([])
  });

  get itemsArray() {
    return this.form.controls.items;
  }

  addItem() {
    this.itemsArray.push(formGroup({
      name: formControl('', Validators.required),
      quantity: formControl(1, [Validators.required, Validators.min(1)]),
      price: formControl(0, [Validators.required, Validators.min(0)])
    }));
  }

  removeItem(index: number) {
    this.itemsArray.removeAt(index);
  }

  getSubtotal(index: number) {
    return computed(() => {
      const item = this.itemsArray.at(index);
      return item.controls.quantity.value() * item.controls.price.value();
    });
  }

  total = computed(() => {
    return this.itemsArray.controls.reduce((sum, item) => {
      return sum + (item.controls.quantity.value() * item.controls.price.value());
    }, 0);
  });
}
```

## Validators with Signals

### Custom Signal Validators

```typescript
import { ValidatorFn, ValidationErrors } from '@angular/forms';
import { Signal } from '@angular/core';

// Validator that uses signals
export function signalMinLength(min: Signal<number>): ValidatorFn {
  return (control) => {
    const minLen = min();
    const value = control.value as string;

    if (value && value.length < minLen) {
      return { minlength: { required: minLen, actual: value.length } };
    }
    return null;
  };
}

// Usage
@Component({...})
export class DynamicValidatorComponent {
  minLength = signal(5);

  name = formControl('', {
    validators: [signalMinLength(this.minLength)]
  });

  increaseMinLength() {
    this.minLength.update(v => v + 1);
  }
}
```

### Cross-Field Signal Validation

```typescript
export function passwordMatchValidator(
  passwordSignal: () => string
): ValidatorFn {
  return (control) => {
    if (control.value !== passwordSignal()) {
      return { passwordMismatch: true };
    }
    return null;
  };
}

@Component({...})
export class PasswordFormComponent {
  password = formControl('', Validators.required);
  confirmPassword = formControl('', {
    validators: [
      Validators.required,
      passwordMatchValidator(() => this.password.value())
    ]
  });

  form = formGroup({
    password: this.password,
    confirmPassword: this.confirmPassword
  });
}
```

## Effects with Forms

```typescript
import { effect } from '@angular/core';

@Component({...})
export class FormEffectsComponent {
  form = formGroup({
    category: formControl(''),
    subcategory: formControl('')
  });

  constructor() {
    // React to category changes
    effect(() => {
      const category = this.form.controls.category.value();
      console.log('Category changed:', category);

      // Reset subcategory when category changes
      this.form.controls.subcategory.setValue('');
      this.loadSubcategories(category);
    });

    // Auto-save on valid changes
    effect(() => {
      if (this.form.valid() && this.form.dirty()) {
        this.autoSave(this.form.value());
      }
    });
  }

  loadSubcategories(category: string) {
    // Load subcategories based on category
  }

  autoSave(data: any) {
    // Save to server
  }
}
```

## Form Methods

```typescript
@Component({...})
export class FormMethodsComponent {
  form = formGroup({
    name: formControl(''),
    email: formControl('')
  });

  // Set single value
  setName() {
    this.form.controls.name.setValue('John');
  }

  // Set entire form
  setFormValue() {
    this.form.setValue({
      name: 'John',
      email: 'john@example.com'
    });
  }

  // Patch partial values
  patchForm() {
    this.form.patchValue({
      name: 'Jane'
    });
  }

  // Reset form
  resetForm() {
    this.form.reset();
  }

  // Reset with values
  resetWithDefaults() {
    this.form.reset({
      name: 'Default Name',
      email: ''
    });
  }

  // Disable/Enable
  toggleDisabled() {
    if (this.form.disabled()) {
      this.form.enable();
    } else {
      this.form.disable();
    }
  }

  // Mark all as touched
  validateForm() {
    this.form.markAllAsTouched();
  }
}
```

## Complete Example

```typescript
interface ProductFormModel {
  name: string;
  description: string;
  price: number;
  category: string;
  tags: string[];
  inStock: boolean;
}

@Component({
  standalone: true,
  imports: [SignalFormsModule, CurrencyPipe],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div class="field">
        <label>Product Name</label>
        <input formControlName="name" />
        @if (form.controls.name.errors()?.['required'] && form.controls.name.touched()) {
          <span class="error">Name is required</span>
        }
      </div>

      <div class="field">
        <label>Description</label>
        <textarea formControlName="description"></textarea>
        <span class="char-count">
          {{ form.controls.description.value().length }} / 500
        </span>
      </div>

      <div class="field">
        <label>Price</label>
        <input formControlName="price" type="number" step="0.01" />
        <span class="preview">{{ form.controls.price.value() | currency }}</span>
      </div>

      <div class="field">
        <label>Category</label>
        <select formControlName="category">
          <option value="">Select category</option>
          @for (cat of categories; track cat.id) {
            <option [value]="cat.id">{{ cat.name }}</option>
          }
        </select>
      </div>

      <div class="field">
        <label>
          <input type="checkbox" formControlName="inStock" />
          In Stock
        </label>
      </div>

      <div formArrayName="tags">
        <label>Tags</label>
        @for (tag of tagsArray.controls; track $index; let i = $index) {
          <div class="tag-input">
            <input [formControlName]="i" placeholder="Tag" />
            <button type="button" (click)="removeTag(i)">×</button>
          </div>
        }
        <button type="button" (click)="addTag()">Add Tag</button>
      </div>

      <div class="form-status">
        <span [class.valid]="form.valid()">
          {{ form.valid() ? 'Form is valid' : 'Please fix errors' }}
        </span>
      </div>

      <div class="actions">
        <button type="button" (click)="resetForm()">Reset</button>
        <button type="submit" [disabled]="form.invalid() || form.pristine()">
          Save Product
        </button>
      </div>
    </form>

    <div class="preview">
      <h3>Preview</h3>
      <pre>{{ form.value() | json }}</pre>
    </div>
  `
})
export class ProductFormComponent {
  categories = [
    { id: 'electronics', name: 'Electronics' },
    { id: 'clothing', name: 'Clothing' },
    { id: 'home', name: 'Home & Garden' }
  ];

  form = formGroup<ProductFormModel>({
    name: formControl('', {
      validators: [Validators.required, Validators.minLength(3)]
    }),
    description: formControl('', {
      validators: [Validators.maxLength(500)]
    }),
    price: formControl(0, {
      validators: [Validators.required, Validators.min(0.01)]
    }),
    category: formControl('', {
      validators: [Validators.required]
    }),
    tags: formArray([formControl('')]),
    inStock: formControl(true)
  });

  get tagsArray() {
    return this.form.controls.tags;
  }

  addTag() {
    this.tagsArray.push(formControl(''));
  }

  removeTag(index: number) {
    this.tagsArray.removeAt(index);
  }

  resetForm() {
    this.form.reset({
      name: '',
      description: '',
      price: 0,
      category: '',
      tags: [''],
      inStock: true
    });
  }

  onSubmit() {
    if (this.form.valid()) {
      const product = this.form.value();
      console.log('Saving product:', product);
      // Save to server
    }
  }
}
```

## Migration from Reactive Forms

| Reactive Forms | Signal Forms |
|----------------|--------------|
| `new FormControl('')` | `formControl('')` |
| `new FormGroup({})` | `formGroup({})` |
| `new FormArray([])` | `formArray([])` |
| `control.value` | `control.value()` |
| `control.valid` | `control.valid()` |
| `control.valueChanges` | `effect(() => control.value())` |
| `control.statusChanges` | `effect(() => control.status())` |

## Summary

| Feature | Description |
|---------|-------------|
| `formControl()` | Signal-based form control |
| `formGroup()` | Signal-based form group |
| `formArray()` | Signal-based form array |
| State signals | `value()`, `valid()`, `touched()`, etc. |
| Typed by default | Full TypeScript support |
| Effect integration | React to form changes |

## Next Steps

- [HTTP Client](./11-http-client.md) - Send form data to APIs
- [Angular Signals](./12-signals.md) - Deep dive into signals

---

[Previous: Reactive Forms](./09-reactive-forms.md) | [Back to Index](./README.md) | [Next: HTTP Client](./11-http-client.md)
