# State Management

Managing state is a crucial aspect of building complex Angular applications. State refers to the data that drives your application's UI and business logic. As applications grow, maintaining and synchronizing this data across various components can become challenging.

Angular provides several approaches to state management, ranging from simple component-level state to sophisticated global state management solutions.

## Component State

For simple scenarios, state can be managed directly within components using properties and signals. This approach is suitable for data that is only relevant to a single component or a small, closely related group of components.

*   **Signals**: Angular's new reactive primitive, `signal()`, `computed()`, and `effect()`, offer a powerful and efficient way to manage local component state. They provide fine-grained reactivity, making it easy to react to changes and update the UI accordingly.
    *   [Learn more about Angular Signals](./12-signals.md)

## Service-Based State

For state that needs to be shared across multiple components or services, creating a dedicated service to hold and manage this state is a common pattern. This promotes reusability and separation of concerns.

*   **Simple Store with Signals**: A service can encapsulate signals, providing methods to update state and exposing read-only signals for consumption. This pattern is effective for mid-sized applications that need shared state without the overhead of a full-fledged state management library.
    *   [Example of Service-Based State Management with Signals in 12-signals.md](./12-signals.md#state-management-with-signals)

## Global State Management Libraries

For large-scale applications with complex state interactions, dedicated state management libraries offer more structured and predictable ways to handle application state.

*   **NgRx Store**: A powerful and opinionated library inspired by Redux, NgRx provides a complete state management solution based on the Redux pattern. It uses a single, immutable store, actions for dispatching changes, reducers for handling state transitions, and effects for side effects. NgRx promotes a clear, predictable, and testable state flow.
    *   [Learn more about NgRx Store](./15-ngrx.md) (Chapter not yet available)

*   **Signal Store**: Built on top of Angular Signals, Signal Store aims to provide a lightweight and modern state management solution that leverages the performance and ergonomics of signals. It's designed to be a simpler alternative to NgRx for applications that prefer a more signal-centric approach.
    *   [Learn more about Signal Store](./16-signal-store.md) (Chapter not yet available)

## Choosing a Strategy

The choice of state management strategy depends on the complexity and scale of your application:

*   **Component State**: Ideal for local component data.
*   **Service-Based State (with Signals)**: Great for sharing state across a few related components, offering good balance between simplicity and maintainability.
*   **NgRx Store**: Best for large, complex applications requiring a highly predictable, scalable, and debuggable state management solution.
*   **Signal Store**: A promising option for applications seeking a modern, signal-native approach to global state.

## Next Steps

- [NgRx Store](./15-ngrx.md) - Learn about Redux-style state management
- [Signal Store](./16-signal-store.md) - Explore signal-based state management

---

[Previous: RxJS & Observables](./13-rxjs.md) | [Back to Index](./README.md) | [Next: NgRx Store](./15-ngrx.md)