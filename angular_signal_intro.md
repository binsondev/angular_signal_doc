# Angular Signals: From Introduction to Angular 19

Angular Signals represent a paradigm shift in how Angular manages state and reactivity. They offer a fine-grained change detection mechanism, moving away from the blanket approach of Zone.js for many use cases, leading to improved performance and a more intuitive developer experience.

Signals were introduced as a developer preview in **Angular 16** and became stable in **Angular 17**.

## Core Signal Primitives

These are the foundational building blocks of the Angular Signals system.

### 1. `signal<T>()` (Writable Signals)

The `signal()` function creates a new writable signal (`WritableSignal<T>`) that holds a value. This value can be read and updated, and any dependents (like `computed` signals or `effect`s) will be notified upon change.

**Description:**
* A `WritableSignal<T>` is a container for a value that can change over time.
* When the value of a writable signal changes, Angular knows exactly which parts of the application depend on this signal and can update them efficiently.

**Key Methods:**
* `set(value: T)`: Directly sets a new value for the signal.
* `update(updateFn: (currentValue: T) => T)`: Updates the signal's value based on its current value.
* `mutate(mutatorFn: (currentValue: T) => void)`: (Available for signals holding mutable objects/arrays) Mutates the current value in place. This is generally discouraged for complex state management in favor of `set` or `update` with immutable patterns, but can be useful for specific scenarios.
* `asReadonly(): Signal<T>`: Returns a read-only version of the signal.

**Code Example:**

```typescript
import { Component, signal, WritableSignal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <button (click)="increment()">Increment</button>
    <button (click)="decrement()">Decrement</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  // Create a writable signal with an initial value of 0
  count: WritableSignal<number> = signal(0);

  increment(): void {
    // Update the signal's value based on the current value
    this.count.update(current => current + 1);
  }

  decrement(): void {
    this.count.update(current => current - 1);
  }

  reset(): void {
    // Set the signal's value directly
    this.count.set(0);
  }
}