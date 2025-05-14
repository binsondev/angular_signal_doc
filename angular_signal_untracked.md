The untracked() function allows you to read a signal's value inside a reactive context (like computed or effect) without creating a dependency on that signal.

Description:

Useful when you need to access a signal's current value but don't want the computed or effect to re-run when that specific signal changes.
Code Example:

import { Component, signal, effect, untracked, WritableSignal, Injector, runInInjectionContext } from '@angular/core';

@Component({
  selector: 'app-untracked-example',
  standalone: true,
  template: `
    <p>Trigger Value: <input #tv (input)="triggerValue.set(+tv.value)" type="number" [value]="triggerValue()" /></p>
    <p>Observed Value: <input #ov (input)="observedValue.set(ov.value)" [value]="observedValue()" /></p>
    <p>Effect Log: {{ effectLog() }}</p>
  `
})
export class UntrackedExampleComponent {
  triggerValue: WritableSignal<number> = signal(0);
  observedValue: WritableSignal<string> = signal('hello');
  effectLog: WritableSignal<string> = signal('');

  constructor(private injector: Injector) {
    runInInjectionContext(this.injector, () => {
      effect(() => {
        // This effect will ONLY re-run when `triggerValue` changes.
        // Changes to `observedValue` will NOT trigger this effect.
        const currentTrigger = this.triggerValue();
        const currentObserved = untracked(() => this.observedValue()); // Read without tracking

        const logMessage = `Effect ran. Trigger: ${currentTrigger}, Observed (untracked): ${currentObserved}`;
        console.log(logMessage);
        this.effectLog.set(logMessage);
      });
    });
  }
}