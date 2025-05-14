output() (Signal Outputs)
The output() function is a signal-based alternative to the @Output() decorator. It returns an OutputEmitterRef<T> which has an emit() method.

Description:

Modern way to declare component outputs.
Can be used with or without signal inputs
Code Example:

import { Component, output, OutputEmitterRef } from '@angular/core';

@Component({
  selector: 'app-action-button',
  standalone: true,
  template: `
    <button (click)="handleClick()">{{ buttonText() }}</button>
  `
})
export class ActionButtonComponent {
  // Define an output that emits a string payload
  action: OutputEmitterRef<string> = output<string>();
  // Can also use input() for buttonText
  buttonText = signal('Click Me');

  handleClick(): void {
    const timestamp = new Date().toLocaleTimeString();
    this.action.emit(`Button clicked at ${timestamp}`);
  }
}

// Parent component usage:
@Component({
  selector: 'app-action-container',
  standalone: true,
  imports: [ActionButtonComponent],
  template: `
    <app-action-button (action)="onAction($event)" />
    <p>Last action: {{ lastAction() }}</p>
  `
})
export class ActionContainerComponent {
  lastAction = signal('No action yet.');

  onAction(payload: string): void {
    this.lastAction.set(payload);
    console.log('Action received from child:', payload);
  }
}