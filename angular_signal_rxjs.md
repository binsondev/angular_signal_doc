RxJS Interoperability
Angular provides utilities in @angular/core/rxjs-interop to bridge Signals and RxJS Observables.

1. toSignal<T>()
Converts an RxJS Observable into a read-only signal (Signal<T | undefined>).

Description:

The signal will always have the most recent value emitted by the Observable.
Handles subscription and unsubscription automatically within an injection context.
Options:
initialValue: Provides an initial value before the Observable emits.
requireSync: If true, asserts that the Observable emits synchronously upon subscription. The signal type won't include undefined.
injector: Explicitly provide an injector if not called in an injection context.
manualCleanup: If true, automatic cleanup is disabled.
Code Example:

import { Component, inject, OnInit } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { HttpClient } from '@angular/common/http';
import { interval, map } from 'rxjs';

interface Post {
  id: number;
  title: string;
  body: string;
}

@Component({
  selector: 'app-data-fetcher',
  standalone: true,
  imports: [], // HttpClientModule needs to be provided in bootstrapApplication or a parent module
  template: `
    <p>Timer Value: {{ timerValue() ?? 'Waiting for timer...' }}</p>
    <div *ngIf="post()">
      <h3>Post Title: {{ post()?.title }}</h3>
      <p>{{ post()?.body }}</p>
    </div>
    <div *ngIf="!post() && !postError()">Loading post...</div>
    <div *ngIf="postError()">Error loading post.</div>
  `
})
export class DataFetcherComponent {
  private http = inject(HttpClient);

  // Convert an interval Observable to a signal
  timerValue = toSignal(interval(1000)); // Emits undefined initially, then 0, 1, 2...

  // Fetch data and convert to signal
  postRequest$ = this.http.get<Post>('[https://jsonplaceholder.typicode.com/posts/1](https://jsonplaceholder.typicode.com/posts/1)');
  post = toSignal(this.postRequest$, { initialValue: null }); // Type Signal<Post | null>

  // Example with requireSync (careful, API must be sync)
  // dataSync = toSignal(of(123), { requireSync: true }); // Signal<number>

  // Example to catch errors
  postError = signal<any>(null);
  anotherPost = toSignal(
    this.http.get<Post>('[https://jsonplaceholder.typicode.com/posts/2').pipe](https://jsonplaceholder.typicode.com/posts/2').pipe)(
      // tap({ error: (err) => this.postError.set(err) }) // Not ideal for toSignal error handling
    ),
    { initialValue: null } // `toSignal` will throw if the observable errors.
                           // Handle errors within the observable or use a more robust pattern.
  );

  constructor() {
    // A more robust way to handle errors with toSignal is often to catch them
    // in the observable pipeline and map them to a success/error state.
    // Or, if the signal itself should reflect the error state, this needs careful setup.
    // The `toSignal` itself doesn't have a built-in error callback for the signal value.
    // If the source observable errors, the signal will also error when read.
  }
}

toObservable<T>()
Converts a signal (Signal<T>) into an RxJS Observable.

Description:

The Observable emits the current value of the signal upon subscription and then emits subsequent values whenever the signal changes.
Options:
injector: Explicitly provide an injector if not called in an injection context.
Code Example:

import { Component, signal, WritableSignal, OnInit, OnDestroy } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, map, Subscription, tap } from 'rxjs';

@Component({
  selector: 'app-signal-to-observable',
  standalone: true,
  template: `
    <input #search (input)="searchTerm.set(search.value)" placeholder="Search..." />
    <p>Debounced search term: {{ debouncedSearchTerm() }}</p>
  `
})
export class SignalToObservableComponent implements OnInit, OnDestroy {
  searchTerm: WritableSignal<string> = signal('');
  debouncedSearchTerm = signal('');

  private searchTerm$: Subscription | undefined;

  constructor() {
    // Convert the searchTerm signal to an Observable
    const searchTermObservable = toObservable(this.searchTerm);

    this.searchTerm$ = searchTermObservable.pipe(
      debounceTime(300), // Wait for 300ms of inactivity
      map(term => term.toUpperCase()),
      tap(debouncedTerm => console.log('Debounced and uppercased:', debouncedTerm))
    ).subscribe(processedTerm => {
      this.debouncedSearchTerm.set(processedTerm);
    });
  }

  ngOnDestroy(): void {
    this.searchTerm$?.unsubscribe();
  }
}