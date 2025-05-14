effect() (Effects for Side Effects)
The effect() function registers a callback that runs whenever one or more signals read within its execution change. Effects are primarily used for side effects that don't directly render UI but need to react to state changes.

Description:

An effect runs at least once upon creation.
It automatically tracks signal dependencies within its callback.
Useful for logging, manual DOM manipulation (though generally prefer template bindings), data synchronization with non-Angular systems, or managing long-lived resources.
Effects run asynchronously during the change detection process.
effect() can take an optional onCleanup function as the first argument to its callback, which is executed before the effect re-runs or when the effect is destroyed
Code Example:

import { Component, signal, effect, WritableSignal, Injector, runInInjectionContext } from '@angular/core';

@Component({
  selector: 'app-logger',
  standalone: true,
  template: `
    <p>Value: <input #val (input)="data.set(val.value)" [value]="data()" /></p>
  `
})
export class LoggerComponent {
  data: WritableSignal<string> = signal('initial value');

  constructor(private injector: Injector) {
    // It's common to run effects within an injection context (e.g., constructor)
    // so they are automatically cleaned up when the component is destroyed.
    runInInjectionContext(this.injector, () => {
      effect((onCleanup) => {
        const currentValue = this.data();
        console.log(`Data changed to: ${currentValue}`);

        const timer = setTimeout(() => {
          console.log(`Effect side-task for: ${currentValue}`);
        }, 1000);

        onCleanup(() => {
          clearTimeout(timer);
          console.log(`Cleaning up effect for old value before re-run or destroy.`);
        });
      });
    });
  }
}