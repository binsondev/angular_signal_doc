computed<T>() (Computed/Derived Signals)
The computed() function creates a read-only signal (Signal<T>) whose value is derived from other signals. It automatically recalculates its value whenever any of its dependent signals change.

Description:

Computed signals are declarative and memoized.
They only recompute when one of their dependencies changes.
They cannot be directly set or updated.
Code Example:
import { Component, signal, computed, Signal, WritableSignal } from '@angular/core';

@Component({
  selector: 'app-derived-values',
  standalone: true,
  template: `
    <p>First Name: <input #fn (input)="firstName.set(fn.value)" [value]="firstName()" /></p>
    <p>Last Name: <input #ln (input)="lastName.set(ln.value)" [value]="lastName()" /></p>
    <p>Full Name: {{ fullName() }}</p>
    <p>Is Full Name Long?: {{ isFullNameLong() ? 'Yes' : 'No' }}</p>
  `
})
export class DerivedValuesComponent {
  firstName: WritableSignal<string> = signal('John');
  lastName: WritableSignal<string> = signal('Doe');

  // Create a computed signal for the full name
  fullName: Signal<string> = computed(() => `${this.firstName()} ${this.lastName()}`);

  // Create another computed signal based on the fullName signal
  isFullNameLong: Signal<boolean> = computed(() => this.fullName().length > 10);
}

When NOT to use effect:

For deriving state: Use computed() instead.
For propagating state changes within Angular: This is usually handled automatically by signals.