# Project: Task Manager App

Build a complete task management application applying all concepts learned in this guide.

## Project Overview

A full-featured task manager with:
- User authentication
- Project and task management
- Real-time updates
- Responsive design with Angular Material

## Setup

```bash
# Create project
ng new task-manager --style=scss --ssr --routing

# Add dependencies
cd task-manager
ng add @angular/material
npm install @ngrx/signals
```

## Project Structure

```
src/app/
├── core/
│   ├── auth/
│   │   ├── auth.service.ts
│   │   ├── auth.guard.ts
│   │   └── auth.interceptor.ts
│   ├── http/
│   │   └── error.interceptor.ts
│   └── services/
│       └── notification.service.ts
├── shared/
│   ├── components/
│   │   ├── confirm-dialog/
│   │   └── loading-spinner/
│   ├── pipes/
│   │   └── relative-time.pipe.ts
│   └── index.ts
├── features/
│   ├── auth/
│   │   ├── login/
│   │   ├── register/
│   │   └── auth.routes.ts
│   ├── dashboard/
│   │   └── dashboard.component.ts
│   ├── projects/
│   │   ├── project-list/
│   │   ├── project-detail/
│   │   ├── project.service.ts
│   │   └── projects.routes.ts
│   └── tasks/
│       ├── task-list/
│       ├── task-form/
│       ├── task.service.ts
│       └── tasks.routes.ts
├── layouts/
│   ├── main-layout/
│   └── auth-layout/
├── app.component.ts
├── app.config.ts
└── app.routes.ts
```

## Models

**models/user.model.ts:**

```typescript
export interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  createdAt: Date;
}

export interface LoginCredentials {
  email: string;
  password: string;
}

export interface RegisterData extends LoginCredentials {
  name: string;
}
```

**models/project.model.ts:**

```typescript
export interface Project {
  id: string;
  name: string;
  description: string;
  color: string;
  ownerId: string;
  memberIds: string[];
  createdAt: Date;
  updatedAt: Date;
}

export type CreateProject = Pick<Project, 'name' | 'description' | 'color'>;
```

**models/task.model.ts:**

```typescript
export type TaskStatus = 'todo' | 'in_progress' | 'review' | 'done';
export type TaskPriority = 'low' | 'medium' | 'high';

export interface Task {
  id: string;
  title: string;
  description: string;
  status: TaskStatus;
  priority: TaskPriority;
  projectId: string;
  assigneeId?: string;
  dueDate?: Date;
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}

export type CreateTask = Pick<Task, 'title' | 'description' | 'priority' | 'projectId' | 'dueDate' | 'tags'>;
```

## Authentication

**core/auth/auth.service.ts:**

```typescript
import { Injectable, signal, computed, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { tap } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);

  private _user = signal<User | null>(null);
  private _token = signal<string | null>(null);

  readonly user = this._user.asReadonly();
  readonly isAuthenticated = computed(() => !!this._token());

  constructor() {
    this.loadFromStorage();
  }

  login(credentials: LoginCredentials) {
    return this.http.post<{ user: User; token: string }>('/api/auth/login', credentials)
      .pipe(tap(response => this.setSession(response)));
  }

  register(data: RegisterData) {
    return this.http.post<{ user: User; token: string }>('/api/auth/register', data)
      .pipe(tap(response => this.setSession(response)));
  }

  logout() {
    this._user.set(null);
    this._token.set(null);
    localStorage.removeItem('auth');
    this.router.navigate(['/login']);
  }

  getToken() {
    return this._token();
  }

  private setSession(response: { user: User; token: string }) {
    this._user.set(response.user);
    this._token.set(response.token);
    localStorage.setItem('auth', JSON.stringify(response));
  }

  private loadFromStorage() {
    const stored = localStorage.getItem('auth');
    if (stored) {
      const { user, token } = JSON.parse(stored);
      this._user.set(user);
      this._token.set(token);
    }
  }
}
```

**core/auth/auth.guard.ts:**

```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';
import { AuthService } from './auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};

export const guestGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (!auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/dashboard']);
};
```

**core/auth/auth.interceptor.ts:**

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const token = auth.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req);
};
```

## Task Store

**features/tasks/task.store.ts:**

```typescript
import { Injectable, signal, computed, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class TaskStore {
  private http = inject(HttpClient);

  // State
  private _tasks = signal<Task[]>([]);
  private _loading = signal(false);
  private _selectedProjectId = signal<string | null>(null);
  private _filter = signal<TaskStatus | 'all'>('all');
  private _searchTerm = signal('');

  // Selectors
  readonly tasks = this._tasks.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly filter = this._filter.asReadonly();
  readonly searchTerm = this._searchTerm.asReadonly();

  readonly filteredTasks = computed(() => {
    let tasks = this._tasks();

    // Filter by project
    const projectId = this._selectedProjectId();
    if (projectId) {
      tasks = tasks.filter(t => t.projectId === projectId);
    }

    // Filter by status
    const status = this._filter();
    if (status !== 'all') {
      tasks = tasks.filter(t => t.status === status);
    }

    // Filter by search
    const term = this._searchTerm().toLowerCase();
    if (term) {
      tasks = tasks.filter(t =>
        t.title.toLowerCase().includes(term) ||
        t.description.toLowerCase().includes(term)
      );
    }

    return tasks;
  });

  readonly tasksByStatus = computed(() => {
    const tasks = this.filteredTasks();
    return {
      todo: tasks.filter(t => t.status === 'todo'),
      in_progress: tasks.filter(t => t.status === 'in_progress'),
      review: tasks.filter(t => t.status === 'review'),
      done: tasks.filter(t => t.status === 'done')
    };
  });

  readonly stats = computed(() => ({
    total: this._tasks().length,
    todo: this._tasks().filter(t => t.status === 'todo').length,
    inProgress: this._tasks().filter(t => t.status === 'in_progress').length,
    done: this._tasks().filter(t => t.status === 'done').length
  }));

  // Actions
  loadTasks(projectId?: string) {
    this._loading.set(true);
    const url = projectId ? `/api/projects/${projectId}/tasks` : '/api/tasks';

    return this.http.get<Task[]>(url).pipe(
      tap(tasks => {
        this._tasks.set(tasks);
        this._loading.set(false);
        if (projectId) this._selectedProjectId.set(projectId);
      })
    );
  }

  createTask(task: CreateTask) {
    return this.http.post<Task>('/api/tasks', task).pipe(
      tap(newTask => {
        this._tasks.update(tasks => [...tasks, newTask]);
      })
    );
  }

  updateTask(id: string, updates: Partial<Task>) {
    return this.http.patch<Task>(`/api/tasks/${id}`, updates).pipe(
      tap(updated => {
        this._tasks.update(tasks =>
          tasks.map(t => t.id === id ? updated : t)
        );
      })
    );
  }

  updateStatus(id: string, status: TaskStatus) {
    return this.updateTask(id, { status });
  }

  deleteTask(id: string) {
    return this.http.delete<void>(`/api/tasks/${id}`).pipe(
      tap(() => {
        this._tasks.update(tasks => tasks.filter(t => t.id !== id));
      })
    );
  }

  setFilter(filter: TaskStatus | 'all') {
    this._filter.set(filter);
  }

  setSearchTerm(term: string) {
    this._searchTerm.set(term);
  }
}
```

## Task Board Component

**features/tasks/task-board/task-board.component.ts:**

```typescript
import { Component, inject } from '@angular/core';
import { CdkDragDrop, DragDropModule, moveItemInArray, transferArrayItem } from '@angular/cdk/drag-drop';
import { TaskStore } from '../task.store';
import { TaskCardComponent } from '../task-card/task-card.component';

@Component({
  selector: 'app-task-board',
  standalone: true,
  imports: [DragDropModule, TaskCardComponent],
  template: `
    <div class="board">
      @for (column of columns; track column.id) {
        <div class="column">
          <h3 class="column-header">
            {{ column.title }}
            <span class="count">{{ getColumnTasks(column.id).length }}</span>
          </h3>

          <div
            class="task-list"
            cdkDropList
            [cdkDropListData]="getColumnTasks(column.id)"
            [id]="column.id"
            [cdkDropListConnectedTo]="columnIds"
            (cdkDropListDropped)="onDrop($event)"
          >
            @for (task of getColumnTasks(column.id); track task.id) {
              <app-task-card
                [task]="task"
                cdkDrag
                [cdkDragData]="task"
              />
            } @empty {
              <div class="empty">No tasks</div>
            }
          </div>
        </div>
      }
    </div>
  `,
  styles: [`
    .board {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 1rem;
      padding: 1rem;
      min-height: calc(100vh - 200px);
    }

    .column {
      background: #f5f5f5;
      border-radius: 8px;
      padding: 1rem;
    }

    .column-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 1rem;
      font-size: 1rem;
      font-weight: 600;
    }

    .count {
      background: #ddd;
      padding: 0.25rem 0.5rem;
      border-radius: 12px;
      font-size: 0.875rem;
    }

    .task-list {
      min-height: 200px;
    }

    .empty {
      text-align: center;
      color: #999;
      padding: 2rem;
    }

    .cdk-drag-preview {
      box-shadow: 0 5px 20px rgba(0,0,0,0.2);
    }

    .cdk-drag-placeholder {
      opacity: 0.5;
    }
  `]
})
export class TaskBoardComponent {
  private taskStore = inject(TaskStore);

  columns = [
    { id: 'todo', title: 'To Do' },
    { id: 'in_progress', title: 'In Progress' },
    { id: 'review', title: 'Review' },
    { id: 'done', title: 'Done' }
  ];

  columnIds = this.columns.map(c => c.id);
  tasksByStatus = this.taskStore.tasksByStatus;

  getColumnTasks(status: string) {
    return this.tasksByStatus()[status as TaskStatus] || [];
  }

  onDrop(event: CdkDragDrop<Task[]>) {
    if (event.previousContainer === event.container) {
      moveItemInArray(event.container.data, event.previousIndex, event.currentIndex);
    } else {
      const task = event.item.data as Task;
      const newStatus = event.container.id as TaskStatus;

      this.taskStore.updateStatus(task.id, newStatus).subscribe();

      transferArrayItem(
        event.previousContainer.data,
        event.container.data,
        event.previousIndex,
        event.currentIndex
      );
    }
  }
}
```

## Task Form

**features/tasks/task-form/task-form.component.ts:**

```typescript
import { Component, inject, input, output } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatDatepickerModule } from '@angular/material/datepicker';
import { MatButtonModule } from '@angular/material/button';
import { MatChipsModule } from '@angular/material/chips';

@Component({
  selector: 'app-task-form',
  standalone: true,
  imports: [
    ReactiveFormsModule,
    MatFormFieldModule,
    MatInputModule,
    MatSelectModule,
    MatDatepickerModule,
    MatButtonModule,
    MatChipsModule
  ],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <mat-form-field appearance="outline" class="full-width">
        <mat-label>Title</mat-label>
        <input matInput formControlName="title" />
        @if (form.get('title')?.hasError('required')) {
          <mat-error>Title is required</mat-error>
        }
      </mat-form-field>

      <mat-form-field appearance="outline" class="full-width">
        <mat-label>Description</mat-label>
        <textarea matInput formControlName="description" rows="4"></textarea>
      </mat-form-field>

      <div class="row">
        <mat-form-field appearance="outline">
          <mat-label>Priority</mat-label>
          <mat-select formControlName="priority">
            <mat-option value="low">Low</mat-option>
            <mat-option value="medium">Medium</mat-option>
            <mat-option value="high">High</mat-option>
          </mat-select>
        </mat-form-field>

        <mat-form-field appearance="outline">
          <mat-label>Due Date</mat-label>
          <input matInput [matDatepicker]="picker" formControlName="dueDate" />
          <mat-datepicker-toggle matIconSuffix [for]="picker" />
          <mat-datepicker #picker />
        </mat-form-field>
      </div>

      <mat-form-field appearance="outline" class="full-width">
        <mat-label>Tags</mat-label>
        <mat-chip-grid #chipGrid>
          @for (tag of tags; track tag) {
            <mat-chip-row (removed)="removeTag(tag)">
              {{ tag }}
              <button matChipRemove>×</button>
            </mat-chip-row>
          }
        </mat-chip-grid>
        <input
          [matChipInputFor]="chipGrid"
          (matChipInputTokenEnd)="addTag($event)"
          placeholder="Add tag..."
        />
      </mat-form-field>

      <div class="actions">
        <button mat-button type="button" (click)="cancelled.emit()">Cancel</button>
        <button mat-raised-button color="primary" type="submit" [disabled]="form.invalid">
          {{ task() ? 'Update' : 'Create' }} Task
        </button>
      </div>
    </form>
  `
})
export class TaskFormComponent {
  private fb = inject(FormBuilder);

  task = input<Task | null>(null);
  projectId = input.required<string>();

  submitted = output<CreateTask>();
  cancelled = output<void>();

  tags: string[] = [];

  form = this.fb.group({
    title: ['', Validators.required],
    description: [''],
    priority: ['medium' as TaskPriority],
    dueDate: [null as Date | null]
  });

  ngOnInit() {
    const task = this.task();
    if (task) {
      this.form.patchValue(task);
      this.tags = [...task.tags];
    }
  }

  addTag(event: any) {
    const value = (event.value || '').trim();
    if (value && !this.tags.includes(value)) {
      this.tags.push(value);
    }
    event.chipInput?.clear();
  }

  removeTag(tag: string) {
    this.tags = this.tags.filter(t => t !== tag);
  }

  onSubmit() {
    if (this.form.valid) {
      this.submitted.emit({
        ...this.form.value as any,
        projectId: this.projectId(),
        tags: this.tags
      });
    }
  }
}
```

## Main Layout

**layouts/main-layout/main-layout.component.ts:**

```typescript
import { Component, inject, signal } from '@angular/core';
import { RouterOutlet, RouterLink, RouterLinkActive } from '@angular/router';
import { MatToolbarModule } from '@angular/material/toolbar';
import { MatSidenavModule } from '@angular/material/sidenav';
import { MatListModule } from '@angular/material/list';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { MatMenuModule } from '@angular/material/menu';
import { AuthService } from '../../core/auth/auth.service';

@Component({
  selector: 'app-main-layout',
  standalone: true,
  imports: [
    RouterOutlet,
    RouterLink,
    RouterLinkActive,
    MatToolbarModule,
    MatSidenavModule,
    MatListModule,
    MatIconModule,
    MatButtonModule,
    MatMenuModule
  ],
  template: `
    <mat-sidenav-container>
      <mat-sidenav #sidenav mode="side" [opened]="!isMobile()">
        <div class="logo">
          <mat-icon>task_alt</mat-icon>
          <span>TaskManager</span>
        </div>

        <mat-nav-list>
          <a mat-list-item routerLink="/dashboard" routerLinkActive="active">
            <mat-icon matListItemIcon>dashboard</mat-icon>
            <span matListItemTitle>Dashboard</span>
          </a>
          <a mat-list-item routerLink="/projects" routerLinkActive="active">
            <mat-icon matListItemIcon>folder</mat-icon>
            <span matListItemTitle>Projects</span>
          </a>
          <a mat-list-item routerLink="/tasks" routerLinkActive="active">
            <mat-icon matListItemIcon>checklist</mat-icon>
            <span matListItemTitle>All Tasks</span>
          </a>
        </mat-nav-list>
      </mat-sidenav>

      <mat-sidenav-content>
        <mat-toolbar color="primary">
          <button mat-icon-button (click)="sidenav.toggle()">
            <mat-icon>menu</mat-icon>
          </button>
          <span class="spacer"></span>

          <button mat-icon-button [matMenuTriggerFor]="userMenu">
            <mat-icon>account_circle</mat-icon>
          </button>
          <mat-menu #userMenu="matMenu">
            <div class="user-info">
              <strong>{{ auth.user()?.name }}</strong>
              <small>{{ auth.user()?.email }}</small>
            </div>
            <mat-divider></mat-divider>
            <button mat-menu-item routerLink="/settings">
              <mat-icon>settings</mat-icon>
              <span>Settings</span>
            </button>
            <button mat-menu-item (click)="auth.logout()">
              <mat-icon>logout</mat-icon>
              <span>Logout</span>
            </button>
          </mat-menu>
        </mat-toolbar>

        <main class="content">
          <router-outlet />
        </main>
      </mat-sidenav-content>
    </mat-sidenav-container>
  `,
  styles: [`
    mat-sidenav-container {
      height: 100vh;
    }

    mat-sidenav {
      width: 250px;
    }

    .logo {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      padding: 1rem;
      font-size: 1.25rem;
      font-weight: 600;
    }

    .spacer {
      flex: 1;
    }

    .content {
      padding: 1rem;
    }

    .user-info {
      padding: 1rem;
      display: flex;
      flex-direction: column;
    }

    .active {
      background: rgba(0, 0, 0, 0.04);
    }
  `]
})
export class MainLayoutComponent {
  auth = inject(AuthService);
  isMobile = signal(window.innerWidth < 768);
}
```

## Routes

**app.routes.ts:**

```typescript
import { Routes } from '@angular/router';
import { authGuard, guestGuard } from './core/auth/auth.guard';

export const routes: Routes = [
  {
    path: '',
    redirectTo: 'dashboard',
    pathMatch: 'full'
  },
  {
    path: '',
    loadComponent: () => import('./layouts/main-layout/main-layout.component')
      .then(m => m.MainLayoutComponent),
    canActivate: [authGuard],
    children: [
      {
        path: 'dashboard',
        loadComponent: () => import('./features/dashboard/dashboard.component')
          .then(m => m.DashboardComponent),
        title: 'Dashboard'
      },
      {
        path: 'projects',
        loadChildren: () => import('./features/projects/projects.routes')
          .then(m => m.PROJECTS_ROUTES)
      },
      {
        path: 'tasks',
        loadChildren: () => import('./features/tasks/tasks.routes')
          .then(m => m.TASKS_ROUTES)
      }
    ]
  },
  {
    path: 'login',
    loadComponent: () => import('./features/auth/login/login.component')
      .then(m => m.LoginComponent),
    canActivate: [guestGuard],
    title: 'Login'
  },
  {
    path: 'register',
    loadComponent: () => import('./features/auth/register/register.component')
      .then(m => m.RegisterComponent),
    canActivate: [guestGuard],
    title: 'Register'
  },
  {
    path: '**',
    loadComponent: () => import('./features/not-found/not-found.component')
      .then(m => m.NotFoundComponent)
  }
];
```

## Running the Project

```bash
# Development
ng serve

# Production build
ng build --configuration=production

# Run tests
ng test

# E2E tests
ng e2e
```

## Summary

This project demonstrates:

- **Component architecture**: Smart/presentational pattern
- **State management**: Signal-based store
- **Routing**: Lazy loading, guards, resolvers
- **Forms**: Reactive forms with validation
- **Material Design**: Consistent UI components
- **Drag & Drop**: Task board interaction
- **Authentication**: JWT-based auth flow
- **Best practices**: Structure, patterns, testing

## Next Steps

- Add real-time updates with WebSockets
- Implement file attachments
- Add notification system
- Deploy to production

---

[Previous: Deployment](./29-deployment.md) | [Back to Index](./README.md)
