model() (Model Inputs for Two-Way Binding)
The model() function creates a writable signal (ModelSignal<T>) that is designed for two-way data binding ([(modelName)]="value"). It simplifies the traditional @Input() and @Output() valueChange pattern.

Description:

Provides a signal-based approach to two-way binding.
The component can update the model signal internally, and these changes are propagated back to the parent.
Parents can also update the signal bound to the model input.
Options:

model<T>(): Optional model (type ModelSignal<T|undefined>).
model<T>(defaultValue: T): Optional model with a default value (type ModelSignal<T>).
model.required<T>(): Required model (type ModelSignal<T>).
{ alias: 'publicName' }: Defines a public alias for the model input (used in the [(publicName)] binding).

Code Example:

import { Component, model, ModelSignal } from '@angular/core';

@Component({
  selector: 'app-custom-input',
  standalone: true,
  template: `
    <div>
      <label>{{ label() }}: </label>
      <input
        [value]="value()"
        (input)="onInput($event)"
      />
      <button (click)="clearValue()">Clear</button>
    </div>
  `
})
export class CustomInputComponent {
  // Required model input
  value: ModelSignal<string> = model.required<string>();
  // Optional input for label
  label: ModelSignal<string | undefined> = model<string>(); // Can also use input() here if not two-way bound

  onInput(event: Event): void {
    const target = event.target as HTMLInputElement;
    this.value.set(target.value); // Update the model signal
  }

  clearValue(): void {
    this.value.set('');
  }
}

// Parent component usage:
@Component({
  selector: 'app-form',
  standalone: true,
  imports: [CustomInputComponent],
  template: `
    <app-custom-input [(value)]="name" label="Your Name" />
    <p>Current Name in Parent: {{ name() }}</p>
    <button (click)="setNameInParent()">Set Name From Parent</button>
  `
})
export class FormComponent {
  name: WritableSignal<string> = signal('Initial Name');

  setNameInParent() {
    this.name.set('Set by Parent');
  }
}