# Enterprise Patterns & Advanced Architecture

Build scalable, maintainable enterprise applications with proven architectural patterns and best practices.

## Overview

Enterprise Angular applications require additional considerations:
- **Scalability**: Handle millions of users
- **Maintainability**: Long-term code sustainability  
- **Performance**: Sub-second response times
- **Security**: Enterprise-grade protection
- **Testing**: Comprehensive test coverage
- **Monitoring**: Real-time observability

## Enterprise Architecture Patterns

### 1. Layered Architecture (N-Tier)

```
┌─────────────────────────────────────┐
│         Presentation Layer          │
│    (Components, UI, Routing)      │
├─────────────────────────────────────┤
│         Business Layer             │
│  (Services, Business Logic,       │
│   Domain Models, Validation)      │
├─────────────────────────────────────┤
│         Data Access Layer         │
│     (Repositories, APIs,         │
│   Caching, Database Mapping)      │
├─────────────────────────────────────┤
│        Infrastructure Layer       │
│  (Logging, Monitoring, Config,   │
│   Security, Email, Storage)       │
└─────────────────────────────────────┘
```

### 2. Domain-Driven Design (DDD)

#### Domain Models

```typescript
// domain/models/user.domain.ts
export class User {
  constructor(
    public readonly id: UserId,
    public readonly email: Email,
    public readonly profile: UserProfile,
    private _permissions: Permission[]
  ) {}

  hasPermission(permission: string): boolean {
    return this._permissions.some(p => p.name === permission);
  }

  grantPermission(permission: Permission): void {
    if (!this.hasPermission(permission.name)) {
      this._permissions.push(permission);
    }
  }

  revokePermission(permissionName: string): void {
    this._permissions = this._permissions.filter(p => p.name !== permissionName);
  }
}

// Value Objects
export class Email {
  constructor(public readonly value: string) {
    if (!this.isValid(value)) {
      throw new Error('Invalid email format');
    }
  }

  private isValid(email: string): boolean {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
}

export class UserId {
  constructor(public readonly value: string) {
    if (!value || value.length < 8) {
      throw new Error('User ID must be at least 8 characters');
    }
  }
}
```

#### Domain Services

```typescript
import { Injectable } from '@angular/core';
// domain/services/auth.domain.service.ts
@Injectable({ providedIn: 'root' })
export class AuthDomainService {
  constructor(
    private userRepository: UserRepository,
    private tokenService: TokenService
  ) {}

  async authenticate(credentials: LoginCredentials): Promise<AuthResult> {
    const user = await this.userRepository.findByEmail(credentials.email);
    
    if (!user) {
      return AuthResult.failed('Invalid credentials');
    }

    const isValidPassword = await this.validatePassword(
      credentials.password, 
      user.passwordHash
    );

    if (!isValidPassword) {
      return AuthResult.failed('Invalid credentials');
    }

    const token = this.tokenService.generate(user);
    
    return AuthResult.success(user, token);
  }

  private async validatePassword(
    plainPassword: string, 
    hashedPassword: string
  ): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
}
```

### 3. CQRS (Command Query Responsibility Segregation)

#### Commands

```typescript
// commands/create-user.command.ts
export class CreateUserCommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly role: UserRole
  ) {}
}

// commands/update-user.command.ts
export class UpdateUserCommand {
  constructor(
    public readonly userId: string,
    public readonly updates: Partial<User>
  ) {}
}
```

#### Command Handlers

```typescript
import { Injectable } from '@angular/core';
// handlers/create-user.command.handler.ts
@Injectable({ providedIn: 'root' })
export class CreateUserCommandHandler {
  constructor(
    private userRepository: UserRepository,
    private eventBus: EventBus,
    private passwordHasher: PasswordHasher
  ) {}

  async handle(command: CreateUserCommand): Promise<User> {
    // Validate business rules
    const existingUser = await this.userRepository.findByEmail(command.email);
    if (existingUser) {
      throw new Error('User with this email already exists');
    }

    // Create domain entity
    const user = User.create({
      id: UserId.generate(),
      email: new Email(command.email),
      profile: new UserProfile(command.name),
      role: command.role
    });

    // Persist
    await this.userRepository.save(user);

    // Publish domain events
    this.eventBus.publish(new UserCreatedEvent(user.id.value));

    return user;
  }
}
```

#### Queries

```typescript
import { Injectable } from '@angular/core';
// queries/get-users.query.ts
export class GetUsersQuery {
  constructor(
    public readonly page: number = 1,
    public readonly pageSize: number = 10,
    public readonly search?: string,
    public readonly role?: UserRole
  ) {}
}

// query-handlers/get-users.query.handler.ts
@Injectable({ providedIn: 'root' })
export class GetUsersQueryHandler {
  constructor(private userRepository: UserRepository) {}

  async handle(query: GetUsersQuery): Promise<PagedResult<UserDto>> {
    const result = await this.userRepository.findMany({
      page: query.page,
      pageSize: query.pageSize,
      search: query.search,
      role: query.role
    });

    return {
      items: result.items.map(user => UserDto.fromDomain(user)),
      total: result.total,
      page: query.page,
      pageSize: query.pageSize
    };
  }
}
```

### 4. Event-Driven Architecture

#### Domain Events

```typescript
// domain/events/user-created.event.ts
export class UserCreatedEvent extends DomainEvent {
  constructor(public readonly userId: string) {
    super('user.created', new Date());
  }
}

// domain/events/user-role-changed.event.ts
export class UserRoleChangedEvent extends DomainEvent {
  constructor(
    public readonly userId: string,
    public readonly oldRole: UserRole,
    public readonly newRole: UserRole
  ) {
    super('user.role.changed', new Date());
  }
}
```

#### Event Handlers

```typescript
import { Injectable } from '@angular/core';
// event-handlers/send-welcome-email.handler.ts
@Injectable({ providedIn: 'root' })
export class SendWelcomeEmailHandler implements EventHandler<UserCreatedEvent> {
  constructor(private emailService: EmailService) {}

  async handle(event: UserCreatedEvent): Promise<void> {
    await this.emailService.sendWelcomeEmail(event.userId);
  }
}

// event-handlers/audit-log.handler.ts
@Injectable({ providedIn: 'root' })
export class AuditLogHandler implements EventHandler<DomainEvent> {
  constructor(private auditService: AuditService) {}

  async handle(event: DomainEvent): Promise<void> {
    await this.auditService.logEvent({
      eventType: event.eventType,
      timestamp: event.timestamp,
      data: event
    });
  }
}
```

## Enterprise Project Structure

```
src/
├── app/
│   ├── core/                          # Core infrastructure
│   │   ├── domain/                    # Domain layer (DDD)
│   │   │   ├── models/                # Domain entities
│   │   │   │   ├── user.domain.ts
│   │   │   │   ├── product.domain.ts
│   │   │   │   └── order.domain.ts
│   │   │   ├── services/              # Domain services
│   │   │   │   ├── auth.domain.service.ts
│   │   │   │   └── pricing.domain.service.ts
│   │   │   ├── events/                # Domain events
│   │   │   │   ├── user-created.event.ts
│   │   │   │   └── order-placed.event.ts
│   │   │   └── value-objects/        # Value objects
│   │   │       ├── email.ts
│   │   │       └── money.ts
│   │   ├── application/               # Application layer
│   │   │   ├── commands/              # Commands
│   │   │   │   ├── create-user.command.ts
│   │   │   │   └── update-order.command.ts
│   │   │   ├── queries/               # Queries
│   │   │   │   ├── get-users.query.ts
│   │   │   │   └── get-orders.query.ts
│   │   │   ├── handlers/             # Command/Query handlers
│   │   │   │   ├── commands/
│   │   │   │   └── queries/
│   │   │   ├── services/              # Application services
│   │   │   │   ├── user.service.ts
│   │   │   │   └── order.service.ts
│   │   │   └── dtos/                  # Data transfer objects
│   │   │       ├── user.dto.ts
│   │   │       └── order.dto.ts
│   │   ├── infrastructure/            # Infrastructure layer
│   │   │   ├── repositories/          # Data access
│   │   │   │   ├── user.repository.ts
│   │   │   │   └── order.repository.ts
│   │   │   ├── events/                # Event handling
│   │   │   │   ├── event-bus.service.ts
│   │   │   │   └── handlers/
│   │   │   ├── external/              # External services
│   │   │   │   ├── email.service.ts
│   │   │   │   └── payment.service.ts
│   │   │   ├── security/              # Security
│   │   │   │   ├── auth.service.ts
│   │   │   │   ├── guards/
│   │   │   │   └── interceptors/
│   │   │   ├── logging/               # Logging
│   │   │   │   ├── logger.service.ts
│   │   │   │   └── error.tracker.ts
│   │   │   ├── caching/               # Caching
│   │   │   │   ├── cache.service.ts
│   │   │   │   └── redis.cache.ts
│   │   │   └── monitoring/            # Monitoring
│   │   │       ├── metrics.service.ts
│   │   │       └── health-check.service.ts
│   │   └── shared/                   # Shared utilities
│   │       ├── constants/
│   │       ├── enums/
│   │       ├── types/
│   │       └── utils/
│   ├── features/                      # Feature modules
│   │   ├── auth/
│   │   │   ├── components/            # UI Components
│   │   │   ├── pages/                 # Page components
│   │   │   ├── state/                 # Feature state
│   │   │   └── routes.ts
│   │   ├── users/
│   │   ├── products/
│   │   └── orders/
│   ├── layouts/                       # Layout components
│   │   ├── main-layout/
│   │   ├── auth-layout/
│   │   └── admin-layout/
│   └── app.config.ts
├── assets/                           # Static assets
├── environments/                    # Environment configs
└── testing/                         # Test utilities
    ├── fixtures/
    ├── mocks/
    └── utils/
```

## Advanced State Management

### NgRx Enterprise Pattern

#### Store Structure

```typescript
// store/index.ts
export interface AppState {
  [fromAuth.AuthFeatureKey]: fromAuth.State;
  [fromUsers.UsersFeatureKey]: fromUsers.State;
  [fromProducts.ProductsFeatureKey]: fromProducts.State;
  [fromOrders.OrdersFeatureKey]: fromOrders.State;
  [fromUI.UIFeatureKey]: fromUI.State;
  [fromRouter.RouterFeatureKey]: fromRouter.RouterReducerState;
}

export const reducers: ActionReducerMap<AppState> = {
  [fromAuth.AuthFeatureKey]: fromAuth.reducer,
  [fromUsers.UsersFeatureKey]: fromUsers.reducer,
  [fromProducts.ProductsFeatureKey]: fromProducts.reducer,
  [fromOrders.OrdersFeatureKey]: fromOrders.reducer,
  [fromUI.UIFeatureKey]: fromUI.reducer,
  router: fromRouter.reducer
};

export const metaReducers: MetaReducer<AppState>[] = [
  fromEnvironment.environment.production ? storeFreeze : fromStore.logMeta
];
```

#### Entity Management

```typescript
// store/users/users.reducer.ts
export interface UsersState extends EntityState<User> {
  selectedUserId: string | null;
  loading: boolean;
  error: string | null;
}

export const usersAdapter: EntityAdapter<User> = createEntityAdapter<User>({
  selectId: user => user.id,
  sortComparer: (a, b) => a.name.localeCompare(b.name)
});

export const initialState: UsersState = usersAdapter.getInitialState({
  selectedUserId: null,
  loading: false,
  error: null
});

export const usersReducer = createReducer(
  initialState,
  
  // Load Users
  on(UsersActions.loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),
  
  on(UsersActions.loadUsersSuccess, (state, { users }) =>
    usersAdapter.setAll(users, {
      ...state,
      loading: false
    })
  ),
  
  on(UsersActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),
  
  // Create User
  on(UsersActions.createUserSuccess, (state, { user }) =>
    usersAdapter.addOne(user, state)
  ),
  
  // Update User
  on(UsersActions.updateUserSuccess, (state, { user }) =>
    usersAdapter.updateOne(
      { id: user.id, changes: user },
      state
    )
  ),
  
  // Delete User
  on(UsersActions.deleteUserSuccess, (state, { userId }) =>
    usersAdapter.removeOne(userId, state)
  )
);

// Selectors
export const { selectAll, selectEntities, selectIds, selectTotal } = 
  usersAdapter.getSelectors();

export const selectUsersState = createFeatureSelector<UsersState>('users');

export const selectAllUsers = createSelector(
  selectUsersState,
  selectAll
);

export const selectUserEntities = createSelector(
  selectUsersState,
  selectEntities
);

export const selectSelectedUserId = createSelector(
  selectUsersState,
  (state: UsersState) => state.selectedUserId
);

export const selectSelectedUser = createSelector(
  selectUserEntities,
  selectSelectedUserId,
  (entities, selectedId) => selectedId ? entities[selectedId] : null
);
```

## Enterprise Security Patterns

### 1. JWT with Refresh Tokens

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { tap, map } from 'rxjs/operators';
// core/infrastructure/security/token.service.ts
@Injectable({ providedIn: 'root' })
export class TokenService {
  private readonly ACCESS_TOKEN_KEY = 'access_token';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';
  
  constructor(private http: HttpClient) {}

  generateTokens(user: User): TokenPair {
    const accessToken = this.generateAccessToken(user);
    const refreshToken = this.generateRefreshToken();
    
    // Store refresh token securely
    this.storeRefreshToken(refreshToken, user.id);
    
    return { accessToken, refreshToken };
  }

  async refreshAccessToken(): Promise<string> {
    const refreshToken = this.getStoredRefreshToken();
    
    if (!refreshToken) {
      throw new Error('No refresh token available');
    }

    return this.http.post<{ accessToken: string }>('/auth/refresh', {
      refreshToken
    }).pipe(
      tap(response => this.setAccessToken(response.accessToken)),
      map(response => response.accessToken)
    ).toPromise();
  }

  private generateAccessToken(user: User): string {
    return jwt.sign(
      {
        sub: user.id,
        email: user.email,
        role: user.role,
        permissions: user.permissions
      },
      environment.jwtSecret,
      { expiresIn: '15m' }
    );
  }

  private generateRefreshToken(): string {
    return crypto.randomUUID() + '-' + Date.now();
  }

  validateAccessToken(token: string): JWTPayload {
    try {
      return jwt.verify(token, environment.jwtSecret) as JWTPayload;
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }
}
```

### 2. Role-Based Access Control (RBAC)

```typescript
import { Injectable, inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
// core/infrastructure/security/rbac.service.ts
@Injectable({ providedIn: 'root' })
export class RBACService {
  constructor(private auth: AuthService) {}

  hasPermission(permission: string): boolean {
    const user = this.auth.currentUser();
    
    if (!user) return false;
    
    return user.permissions.includes(permission) ||
           this.hasRolePermission(user.role, permission);
  }

  hasAnyPermission(permissions: string[]): boolean {
    return permissions.some(permission => this.hasPermission(permission));
  }

  hasAllPermissions(permissions: string[]): boolean {
    return permissions.every(permission => this.hasPermission(permission));
  }

  canAccessResource(resource: string, action: string): boolean {
    const requiredPermission = `${resource}.${action}`;
    return this.hasPermission(requiredPermission);
  }

  private hasRolePermission(role: UserRole, permission: string): boolean {
    const rolePermissions = ROLE_PERMISSIONS[role];
    return rolePermissions?.includes(permission) || false;
  }
}

// Permission Guard
export const permissionGuard = (permission: string): CanActivateFn => {
  return () => {
    const rbac = inject(RBACService);
    
    if (rbac.hasPermission(permission)) {
      return true;
    }
    
    return inject(Router).createUrlTree(['/unauthorized']);
  };
};

// Usage in routes
{
  path: 'admin',
  canActivate: [permissionGuard('admin.access')]
}
```

## Enterprise Error Handling

### Global Error Handler with Monitoring

```typescript
import { Injectable } from '@angular/core';
import { HttpErrorResponse } from '@angular/common/http';
// core/infrastructure/logging/global-error.handler.ts
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(
    private logger: LoggerService,
    private notificationService: NotificationService,
    private errorTracker: ErrorTrackingService
  ) {}

  handleError(error: Error): void {
    // Log error
    this.logger.error('Unhandled error occurred', {
      error: error.message,
      stack: error.stack,
      timestamp: new Date(),
      userAgent: navigator.userAgent,
      url: window.location.href
    });

    // Track in monitoring system
    this.errorTracker.trackError(error, {
      context: this.getErrorContext(error),
      severity: this.getErrorSeverity(error)
    });

    // Show user-friendly message
    const userMessage = this.getUserFriendlyMessage(error);
    this.notificationService.showError(userMessage);

    // Report critical errors
    if (this.isCriticalError(error)) {
      this.reportCriticalError(error);
    }
  }

  private getErrorContext(error: Error): any {
    // Extract relevant context for debugging
    return {
      type: error.constructor.name,
      message: error.message,
      stack: error.stack?.split('\n').slice(0, 5), // First 5 lines
      timestamp: new Date().toISOString(),
      path: window.location.pathname
    };
  }

  private getErrorSeverity(error: Error): 'low' | 'medium' | 'high' | 'critical' {
    if (error instanceof HttpErrorResponse) {
      if (error.status >= 500) return 'critical';
      if (error.status >= 400) return 'medium';
    }
    
    return error.message.includes('network') ? 'high' : 'medium';
  }

  private getUserFriendlyMessage(error: Error): string {
    if (error instanceof HttpErrorResponse) {
      switch (error.status) {
        case 401: return 'Please log in to continue';
        case 403: return 'You don\'t have permission to access this resource';
        case 404: return 'The requested resource was not found';
        case 500: return 'Server error occurred. Please try again later';
        default: return 'An error occurred. Please try again';
      }
    }
    
    return 'An unexpected error occurred. Please try again';
  }
}
```

## Enterprise Caching Strategy

### Multi-Level Caching

```typescript
import { Injectable, inject } from '@angular/core';
// core/infrastructure/caching/cache.service.ts
@Injectable({ providedIn: 'root' })
export class CacheService {
  private memoryCache = new Map<string, CacheEntry>();
  private redisCache: RedisCache;

  constructor() {
    // Cleanup expired entries periodically
    setInterval(() => this.cleanupExpired(), 60000); // Every minute
  }

  async get<T>(key: string): Promise<T | null> {
    // Check memory cache first
    const memoryEntry = this.memoryCache.get(key);
    if (memoryEntry && !this.isExpired(memoryEntry)) {
      return memoryEntry.data;
    }

    // Check Redis cache
    try {
      const redisData = await this.redisCache.get<T>(key);
      if (redisData) {
        // Cache in memory for faster access
        this.memoryCache.set(key, {
          data: redisData,
          expiresAt: Date.now() + 300000 // 5 minutes
        });
        return redisData;
      }
    } catch (error) {
      console.warn('Redis cache unavailable, falling back to memory only');
    }

    return null;
  }

  async set<T>(
    key: string, 
    data: T, 
    options: CacheOptions = {}
  ): Promise<void> {
    const { 
      ttl = 3600, // 1 hour default
      memoryOnly = false 
    } = options;

    // Cache in memory
    this.memoryCache.set(key, {
      data,
      expiresAt: Date.now() + (ttl * 1000)
    });

    // Cache in Redis if not memory-only
    if (!memoryOnly) {
      try {
        await this.redisCache.set(key, data, { ttl });
      } catch (error) {
        console.warn('Failed to cache in Redis:', error);
      }
    }
  }

  async invalidate(pattern: string): Promise<void> {
    // Invalidate memory cache
    for (const key of this.memoryCache.keys()) {
      if (key.match(pattern)) {
        this.memoryCache.delete(key);
      }
    }

    // Invalidate Redis cache
    try {
      await this.redisCache.invalidate(pattern);
    } catch (error) {
      console.warn('Failed to invalidate Redis cache:', error);
    }
  }

  private cleanupExpired(): void {
    for (const [key, entry] of this.memoryCache.entries()) {
      if (this.isExpired(entry)) {
        this.memoryCache.delete(key);
      }
    }
  }

  private isExpired(entry: CacheEntry): boolean {
    return Date.now() > entry.expiresAt;
  }
}

// Cache Decorator
export function Cacheable(options: CacheOptions = {}) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    const cacheService = inject(CacheService);

    descriptor.value = async function (...args: any[]) {
      const cacheKey = `${target.constructor.name}.${propertyKey}:${JSON.stringify(args)}`;
      
      // Try to get from cache
      const cached = await cacheService.get(cacheKey);
      if (cached !== null) {
        return cached;
      }

      // Execute and cache result
      const result = await originalMethod.apply(this, args);
      await cacheService.set(cacheKey, result, options);
      
      return result;
    };

    return descriptor;
  };
}

// Usage
@Injectable({ providedIn: 'root' })
export class UserService {
  @Cacheable({ ttl: 1800 }) // 30 minutes
  async getUserById(id: string): Promise<User> {
    return this.http.get<User>(`/api/users/${id}`).toPromise();
  }
}
```

## Enterprise Monitoring

### Application Metrics

```typescript
import { Injectable, inject } from '@angular/core';
// core/infrastructure/monitoring/metrics.service.ts
@Injectable({ providedIn: 'root' })
export class MetricsService {
  private metrics: Map<string, Metric> = new Map();
  private timers: Map<string, number> = new Map();

  constructor() {
    // Send metrics periodically
    setInterval(() => this.flushMetrics(), 30000); // Every 30 seconds
  }

  // Counter metrics
  increment(name: string, tags: Record<string, string> = {}): void {
    const key = this.buildKey(name, tags);
    const metric = this.metrics.get(key) || new CounterMetric(name, tags);
    metric.increment();
    this.metrics.set(key, metric);
  }

  // Histogram metrics (for response times, etc.)
  record(name: string, value: number, tags: Record<string, string> = {}): void {
    const key = this.buildKey(name, tags);
    const metric = this.metrics.get(key) || new HistogramMetric(name, tags);
    metric.record(value);
    this.metrics.set(key, metric);
  }

  // Gauge metrics (for current values)
  gauge(name: string, value: number, tags: Record<string, string> = {}): void {
    const key = this.buildKey(name, tags);
    const metric = this.metrics.get(key) || new GaugeMetric(name, tags);
    metric.set(value);
    this.metrics.set(key, metric);
  }

  // Timer helpers
  startTimer(name: string): void {
    this.timers.set(name, Date.now());
  }

  endTimer(name: string, tags: Record<string, string> = {}): void {
    const startTime = this.timers.get(name);
    if (startTime) {
      const duration = Date.now() - startTime;
      this.record(`${name}_duration`, duration, tags);
      this.timers.delete(name);
    }
  }

  // Decorator for method timing
  static Timer(name?: string) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      const originalMethod = descriptor.value;
      const metrics = inject(MetricsService);
      const timerName = name || `${target.constructor.name}.${propertyKey}`;

      descriptor.value = async function (...args: any[]) {
        metrics.startTimer(timerName);
        try {
          const result = await originalMethod.apply(this, args);
          metrics.increment(`${timerName}_success`, { method: propertyKey });
          return result;
        } catch (error) {
          metrics.increment(`${timerName}_error`, { 
            method: propertyKey, 
            error: error.constructor.name 
          });
          throw error;
        } finally {
          metrics.endTimer(timerName, { method: propertyKey });
        }
      };

      return descriptor;
    };
  }

  private async flushMetrics(): Promise<void> {
    if (this.metrics.size === 0) return;

    const metricsData = Array.from(this.metrics.values()).map(metric => metric.serialize());
    
    try {
      await this.sendToMonitoringSystem(metricsData);
      this.metrics.clear();
    } catch (error) {
      console.error('Failed to send metrics:', error);
    }
  }

  private buildKey(name: string, tags: Record<string, string>): string {
    const tagString = Object.entries(tags)
      .sort(([a], [b]) => a.localeCompare(b))
      .map(([k, v]) => `${k}=${v}`)
      .join(',');
    return `${name}{${tagString}}`;
  }
}

// Usage
@Injectable({ providedIn: 'root' })
export class OrderService {
  @MetricsService.Timer('order_processing')
  async processOrder(orderId: string): Promise<Order> {
    const metrics = inject(MetricsService);
    
    try {
      metrics.increment('orders_started');
      const order = await this.fetchOrder(orderId);
      
      metrics.gauge('order_amount', order.total, { currency: order.currency });
      metrics.record('order_items_count', order.items.length);
      
      metrics.increment('orders_completed');
      return order;
    } catch (error) {
      metrics.increment('orders_failed', { error: error.constructor.name });
      throw error;
    }
  }
}
```

## Summary

| Pattern | Use Case | Benefits |
|---------|----------|----------|
| **Layered Architecture** | Large enterprise applications | Clear separation of concerns |
| **DDD** | Complex business domains | Better alignment with business |
| **CQRS** | High-performance systems | Optimized reads/writes |
| **Event-Driven** | Distributed systems | Loose coupling, scalability |
| **Multi-level Caching** | Performance-critical apps | Faster response times |
| **RBAC** | Enterprise security | Granular access control |
| **Metrics & Monitoring** | Production systems | Real-time observability |

## Next Steps

- [Deployment](./29-deployment.md) - Production rollout
- [Project: Task Manager App](./30-project-task-manager.md) - Full end-to-end example

---

[Previous: Architecture](./28-architecture.md) | [Back to Index](./README.md)
