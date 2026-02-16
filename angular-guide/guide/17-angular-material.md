# Angular Material

Angular Material is a UI component library implementing Google's Material Design. It provides high-quality, accessible components.

## Installation

```bash
ng add @angular/material
```

**Prompts:**

| Prompt | Recommended |
|--------|-------------|
| Theme | Custom or Indigo/Pink |
| Typography | Yes |
| Animations | Include and enable |

## Setup

### Import Components

```typescript
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatInputModule } from '@angular/material/input';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatCardModule } from '@angular/material/card';

@Component({
  standalone: true,
  imports: [
    MatButtonModule,
    MatInputModule,
    MatFormFieldModule,
    MatCardModule
  ],
  template: `...`
})
export class MyComponent {}
```

### Theme Configuration

**styles.scss:**

```scss
@use '@angular/material' as mat;

// Include core styles
@include mat.core();

// Define custom theme
$my-primary: mat.m2-define-palette(mat.$m2-indigo-palette);
$my-accent: mat.m2-define-palette(mat.$m2-pink-palette, A200, A100, A400);
$my-warn: mat.m2-define-palette(mat.$m2-red-palette);

$my-theme: mat.m2-define-light-theme((
  color: (
    primary: $my-primary,
    accent: $my-accent,
    warn: $my-warn,
  ),
  typography: mat.m2-define-typography-config(),
  density: 0,
));

// Apply theme
@include mat.all-component-themes($my-theme);

// Dark theme
.dark-theme {
  $dark-theme: mat.m2-define-dark-theme((
    color: (
      primary: $my-primary,
      accent: $my-accent,
      warn: $my-warn,
    ),
  ));
  @include mat.all-component-colors($dark-theme);
}
```

## Common Components

### Buttons

```typescript
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
```

```html
<!-- Basic buttons -->
<button mat-button>Basic</button>
<button mat-raised-button>Raised</button>
<button mat-flat-button>Flat</button>
<button mat-stroked-button>Stroked</button>

<!-- With color -->
<button mat-raised-button color="primary">Primary</button>
<button mat-raised-button color="accent">Accent</button>
<button mat-raised-button color="warn">Warn</button>

<!-- Icon buttons -->
<button mat-icon-button>
  <mat-icon>favorite</mat-icon>
</button>

<!-- FAB -->
<button mat-fab>
  <mat-icon>add</mat-icon>
</button>
<button mat-mini-fab color="accent">
  <mat-icon>edit</mat-icon>
</button>

<!-- With loading state -->
<button mat-raised-button [disabled]="loading()">
  @if (loading()) {
    <mat-spinner diameter="20"></mat-spinner>
  } @else {
    Submit
  }
</button>
```

### Form Fields

```typescript
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatCheckboxModule } from '@angular/material/checkbox';
import { MatRadioModule } from '@angular/material/radio';
import { MatDatepickerModule } from '@angular/material/datepicker';
```

```html
<!-- Text input -->
<mat-form-field appearance="outline">
  <mat-label>Name</mat-label>
  <input matInput [(ngModel)]="name" required>
  <mat-hint>Enter your full name</mat-hint>
  <mat-error>Name is required</mat-error>
</mat-form-field>

<!-- Textarea -->
<mat-form-field appearance="fill">
  <mat-label>Description</mat-label>
  <textarea matInput rows="4"></textarea>
</mat-form-field>

<!-- Select -->
<mat-form-field>
  <mat-label>Country</mat-label>
  <mat-select [(ngModel)]="country">
    @for (c of countries; track c.code) {
      <mat-option [value]="c.code">{{ c.name }}</mat-option>
    }
  </mat-select>
</mat-form-field>

<!-- Checkbox -->
<mat-checkbox [(ngModel)]="agree">I agree to terms</mat-checkbox>

<!-- Radio buttons -->
<mat-radio-group [(ngModel)]="size">
  <mat-radio-button value="S">Small</mat-radio-button>
  <mat-radio-button value="M">Medium</mat-radio-button>
  <mat-radio-button value="L">Large</mat-radio-button>
</mat-radio-group>

<!-- Datepicker -->
<mat-form-field>
  <mat-label>Choose a date</mat-label>
  <input matInput [matDatepicker]="picker" [(ngModel)]="date">
  <mat-datepicker-toggle matIconSuffix [for]="picker"></mat-datepicker-toggle>
  <mat-datepicker #picker></mat-datepicker>
</mat-form-field>
```

### Cards

```typescript
import { MatCardModule } from '@angular/material/card';
```

```html
<mat-card>
  <mat-card-header>
    <mat-card-title>Product Name</mat-card-title>
    <mat-card-subtitle>Category</mat-card-subtitle>
  </mat-card-header>

  <img mat-card-image src="product.jpg" alt="Product">

  <mat-card-content>
    <p>Product description goes here.</p>
  </mat-card-content>

  <mat-card-actions align="end">
    <button mat-button>Share</button>
    <button mat-raised-button color="primary">Buy</button>
  </mat-card-actions>
</mat-card>
```

### Tables

```typescript
import { MatTableModule } from '@angular/material/table';
import { MatSortModule } from '@angular/material/sort';
import { MatPaginatorModule } from '@angular/material/paginator';
```

```typescript
import { Component, ViewChild, OnInit, AfterViewInit } from '@angular/core';
import { MatSort } from '@angular/material/sort';
import { MatPaginator } from '@angular/material/paginator';
@Component({
  imports: [MatTableModule, MatSortModule, MatPaginatorModule],
  template: `
    <table mat-table [dataSource]="dataSource" matSort>
      <!-- Name Column -->
      <ng-container matColumnDef="name">
        <th mat-header-cell *matHeaderCellDef mat-sort-header>Name</th>
        <td mat-cell *matCellDef="let element">{{ element.name }}</td>
      </ng-container>

      <!-- Email Column -->
      <ng-container matColumnDef="email">
        <th mat-header-cell *matHeaderCellDef>Email</th>
        <td mat-cell *matCellDef="let element">{{ element.email }}</td>
      </ng-container>

      <!-- Actions Column -->
      <ng-container matColumnDef="actions">
        <th mat-header-cell *matHeaderCellDef>Actions</th>
        <td mat-cell *matCellDef="let element">
          <button mat-icon-button (click)="edit(element)">
            <mat-icon>edit</mat-icon>
          </button>
          <button mat-icon-button color="warn" (click)="delete(element)">
            <mat-icon>delete</mat-icon>
          </button>
        </td>
      </ng-container>

      <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
      <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
    </table>

    <mat-paginator
      [pageSizeOptions]="[5, 10, 25]"
      showFirstLastButtons>
    </mat-paginator>
  `
})
export class TableComponent implements OnInit, AfterViewInit {
  displayedColumns = ['name', 'email', 'actions'];
  dataSource = new MatTableDataSource<User>([]);

  @ViewChild(MatSort) sort!: MatSort;
  @ViewChild(MatPaginator) paginator!: MatPaginator;

  ngAfterViewInit() {
    this.dataSource.sort = this.sort;
    this.dataSource.paginator = this.paginator;
  }

  applyFilter(event: Event) {
    const filterValue = (event.target as HTMLInputElement).value;
    this.dataSource.filter = filterValue.trim().toLowerCase();
  }
}
```

### Dialogs

```typescript
import { Component, inject } from '@angular/core';
import { MatDialogModule, MatDialog, MatDialogRef, MAT_DIALOG_DATA } from '@angular/material/dialog';
import { MatButtonModule } from '@angular/material/button';

// Dialog component
@Component({
  selector: 'app-confirm-dialog',
  standalone: true,
  imports: [MatDialogModule, MatButtonModule],
  template: `
    <h2 mat-dialog-title>{{ data.title }}</h2>
    <mat-dialog-content>
      <p>{{ data.message }}</p>
    </mat-dialog-content>
    <mat-dialog-actions align="end">
      <button mat-button mat-dialog-close>Cancel</button>
      <button mat-raised-button color="primary" [mat-dialog-close]="true">
        Confirm
      </button>
    </mat-dialog-actions>
  `
})
export class ConfirmDialogComponent {
  data = inject<{ title: string; message: string }>(MAT_DIALOG_DATA);
}

// Usage
@Component({...})
export class ParentComponent {
  private dialog = inject(MatDialog);

  openConfirmDialog() {
    const dialogRef = this.dialog.open(ConfirmDialogComponent, {
      width: '400px',
      data: {
        title: 'Confirm Delete',
        message: 'Are you sure you want to delete this item?'
      }
    });

    dialogRef.afterClosed().subscribe(result => {
      if (result) {
        this.deleteItem();
      }
    });
  }
}
```

### Snackbar

```typescript
import { Component, inject } from '@angular/core';
import { MatSnackBar, MatSnackBarModule } from '@angular/material/snack-bar';

@Component({
  imports: [MatSnackBarModule],
  template: `<button mat-button (click)="showMessage()">Show Message</button>`
})
export class NotificationComponent {
  private snackBar = inject(MatSnackBar);

  showMessage() {
    this.snackBar.open('Item saved successfully!', 'Close', {
      duration: 3000,
      horizontalPosition: 'end',
      verticalPosition: 'top',
      panelClass: ['success-snackbar']
    });
  }

  showError() {
    this.snackBar.open('Error occurred', 'Retry', {
      duration: 5000,
      panelClass: ['error-snackbar']
    }).onAction().subscribe(() => {
      this.retry();
    });
  }
}
```

### Navigation

```typescript
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { MatMenuModule } from '@angular/material/menu';
```

```html
<mat-sidenav-container>
  <mat-sidenav #sidenav mode="side" [opened]="!isMobile()">
    <mat-nav-list>
      <a mat-list-item routerLink="/dashboard" routerLinkActive="active">
        <mat-icon matListItemIcon>dashboard</mat-icon>
        <span matListItemTitle>Dashboard</span>
      </a>
      <a mat-list-item routerLink="/users" routerLinkActive="active">
        <mat-icon matListItemIcon>people</mat-icon>
        <span matListItemTitle>Users</span>
      </a>
      <a mat-list-item routerLink="/settings" routerLinkActive="active">
        <mat-icon matListItemIcon>settings</mat-icon>
        <span matListItemTitle>Settings</span>
      </a>
    </mat-nav-list>
  </mat-sidenav>

  <mat-sidenav-content>
    <mat-toolbar color="primary">
      <button mat-icon-button (click)="sidenav.toggle()">
        <mat-icon>menu</mat-icon>
      </button>
      <span>My App</span>
      <span class="spacer"></span>

      <button mat-icon-button [matMenuTriggerFor]="userMenu">
        <mat-icon>account_circle</mat-icon>
      </button>
      <mat-menu #userMenu="matMenu">
        <button mat-menu-item routerLink="/profile">
          <mat-icon>person</mat-icon>
          <span>Profile</span>
        </button>
        <button mat-menu-item (click)="logout()">
          <mat-icon>logout</mat-icon>
          <span>Logout</span>
        </button>
      </mat-menu>
    </mat-toolbar>

    <router-outlet />
  </mat-sidenav-content>
</mat-sidenav-container>
```

### Tabs

```typescript
import { MatTabsModule } from '@angular/material/tabs';
```

```html
<mat-tab-group>
  <mat-tab label="Overview">
    <div class="tab-content">
      <h3>Overview Content</h3>
    </div>
  </mat-tab>
  <mat-tab label="Details">
    <div class="tab-content">
      <h3>Details Content</h3>
    </div>
  </mat-tab>
  <mat-tab label="Reviews">
    <ng-template matTabContent>
      <!-- Lazy loaded -->
      <app-reviews />
    </ng-template>
  </mat-tab>
</mat-tab-group>
```

### Progress Indicators

```typescript
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';
import { MatProgressBarModule } from '@angular/material/progress-bar';
```

```html
<!-- Spinner -->
<mat-spinner></mat-spinner>
<mat-spinner diameter="40"></mat-spinner>
<mat-spinner mode="determinate" [value]="progress()"></mat-spinner>

<!-- Progress bar -->
<mat-progress-bar mode="indeterminate"></mat-progress-bar>
<mat-progress-bar mode="determinate" [value]="progress()"></mat-progress-bar>
<mat-progress-bar mode="buffer" [value]="progress()" [bufferValue]="bufferProgress()"></mat-progress-bar>
```

### Chips

```typescript
import { MatChipsModule } from '@angular/material/chips';
```

```html
<mat-chip-listbox [(ngModel)]="selectedChips" multiple>
  @for (chip of chips; track chip) {
    <mat-chip-option [value]="chip">{{ chip }}</mat-chip-option>
  }
</mat-chip-listbox>

<!-- Removable chips -->
<mat-chip-set>
  @for (tag of tags(); track tag) {
    <mat-chip (removed)="removeTag(tag)">
      {{ tag }}
      <button matChipRemove>
        <mat-icon>cancel</mat-icon>
      </button>
    </mat-chip>
  }
</mat-chip-set>
```

### Autocomplete

```typescript
import { MatAutocompleteModule } from '@angular/material/autocomplete';
```

```typescript
import { Component, computed, signal } from '@angular/core';
import { FormControl, ReactiveFormsModule } from '@angular/forms';
@Component({
  imports: [MatAutocompleteModule, MatFormFieldModule, MatInputModule, ReactiveFormsModule],
  template: `
    <mat-form-field>
      <mat-label>Country</mat-label>
      <input matInput
             [formControl]="countryControl"
             [matAutocomplete]="auto">
      <mat-autocomplete #auto="matAutocomplete" [displayWith]="displayFn">
        @for (option of filteredCountries(); track option.code) {
          <mat-option [value]="option">{{ option.name }}</mat-option>
        }
      </mat-autocomplete>
    </mat-form-field>
  `
})
export class AutocompleteComponent {
  countryControl = new FormControl('');
  countries = signal<Country[]>([]);

  filteredCountries = computed(() => {
    const value = this.countryControl.value;
    const filterValue = typeof value === 'string' ? value.toLowerCase() : '';
    return this.countries().filter(c =>
      c.name.toLowerCase().includes(filterValue)
    );
  });

  displayFn(country: Country): string {
    return country?.name ?? '';
  }
}
```

## Theming

### Custom Theme

```scss
@use '@angular/material' as mat;

// Define palettes
$custom-primary: (
  50: #e3f2fd,
  100: #bbdefb,
  200: #90caf9,
  // ... more shades
  500: #2196f3,
  700: #1976d2,
  900: #0d47a1,
  contrast: (
    50: rgba(black, 0.87),
    500: white,
    900: white,
  )
);

$my-primary: mat.m2-define-palette($custom-primary);
$my-accent: mat.m2-define-palette(mat.$m2-amber-palette, A200, A100, A400);

$my-theme: mat.m2-define-light-theme((
  color: (
    primary: $my-primary,
    accent: $my-accent,
  )
));

@include mat.all-component-themes($my-theme);
```

### Component-Level Theming

```scss
@use '@angular/material' as mat;

.my-component {
  @include mat.button-theme($my-theme);
  @include mat.card-theme($my-theme);
}
```

## Summary

| Component | Import |
|-----------|--------|
| Button | `MatButtonModule` |
| Input | `MatInputModule`, `MatFormFieldModule` |
| Select | `MatSelectModule` |
| Table | `MatTableModule`, `MatSortModule`, `MatPaginatorModule` |
| Dialog | `MatDialogModule` |
| Snackbar | `MatSnackBarModule` |
| Toolbar | `MatToolbarModule` |
| Sidenav | `MatSidenavModule` |
| Menu | `MatMenuModule` |
| Tabs | `MatTabsModule` |
| Card | `MatCardModule` |
| Progress | `MatProgressSpinnerModule`, `MatProgressBarModule` |

## Next Steps

- [Libraries and Integrations](./18-libraries-and-integrations.md) - Explore popular third-party libraries
- [PrimeNG](./19-primeng.md) - Alternative UI library

---

[Previous: Signal Store](./16-signal-store.md) | [Back to Index](./README.md) | [Next: Libraries and Integrations](./18-libraries-and-integrations.md)
