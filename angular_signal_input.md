input() (Signal Inputs)
The input() function is a replacement for the @Input() decorator, exposing component inputs as read-only signals (InputSignal<T>).

Description:

Provides better type safety (e.g., for required inputs).
Integrates seamlessly with other signal features like computed and effect.
Automatically marks OnPush components as dirty when the input signal changes.
Options:

input<T>(): Optional input (type InputSignal<T|undefined>).
input<T>(defaultValue: T): Optional input with a default value (type InputSignal<T>).
input.required<T>(): Required input (type InputSignal<T>).
{ alias: 'publicName' }: Defines a public alias for the input.
{ transform: (value: U) => T }: Transforms the input value.

Code Example:

import { Component, input, computed, Signal, InputSignal } from '@angular/core';

@Component({
  selector: 'app-user-profile',
  standalone: true,
  template: `
    <div>
      <h2>User Profile</h2>
      <p>ID: {{ userId() }}</p>
      <p>Name: {{ name() }} ({{ nameType() }})</p>
      <p>Status: {{ isActive() ? 'Active' : 'Inactive' }}</p>
      <p>Transformed Input (alias 'userAge'): {{ age() }}</p>
    </div>
  `
})
export class UserProfileComponent {
  // Required input
  userId: InputSignal<string> = input.required<string>();

  // Optional input with a default value
  name: InputSignal<string> = input('Guest');

  // Optional input that can be boolean or string, transformed to boolean
  isActive: InputSignal<boolean> = input(false, {
    transform: (value: boolean | string) => typeof value === 'string' ? value === '' || value === 'true' : !!value
  });

  // Input with an alias and a transform function
  age: InputSignal<string> = input(0, {
    alias: 'userAge',
    transform: (value: number | string) => `Approximately ${value} years`
  });

  nameType: Signal<string> = computed(() => this.name() === 'Guest' ? 'Default User' : 'Registered User');

  constructor() {
    effect(() => {
      console.log(`User ID changed in UserProfileComponent: ${this.userId()}`);
    });
  }
}

// Parent component usage:
@Component({
  selector: 'app-parent-for-profile',
  standalone: true,
  imports: [UserProfileComponent],
  template: `
    <app-user-profile [userId]="currentUserId()" name="Alice" isActive [userAge]="30" />
    <app-user-profile [userId]="'user-002'" isActive="false" [userAge]="'42'" />
    <button (click)="changeUser()">Change User</button>
  `
})
export class ParentForProfileComponent {
  currentUserId = signal('user-001');
  changeUser() {
    this.currentUserId.set('user-xyz-' + Math.random().toString(16).slice(2));
  }
}