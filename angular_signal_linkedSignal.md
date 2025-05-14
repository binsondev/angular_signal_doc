linkedSignal: A Writable, Derived Signal
Explanation
A linkedSignal is a special type of writable signal (WritableSignal) in Angular. Its core purpose is to manage a local piece of state that is derived from other signals (sources) but can also be updated directly.

Here's a breakdown of its key characteristics:

Derived Value: Like a computed() signal, a linkedSignal calculates its initial value and subsequent updates based on one or more source signals. When any of these source signals change, the linkedSignal automatically recomputes its value.
Writable: This is the crucial difference from computed(). While computed() signals are read-only, a linkedSignal is a WritableSignal. This means you can directly change its value using methods like .set() or .update(), independently of its derivation logic.   
Synchronization:
Source to Linked: If a source signal changes, the linkedSignal updates.
Direct Update to Linked: If you directly set() the linkedSignal, it takes that new value. The next time a source signal changes, the derivation logic will re-run and might overwrite your direct update unless the derivation logic itself accounts for such scenarios or if the direct set is meant to be an override until the next source-driven computation. The exact behavior of this interaction (how a direct set is reconciled with a subsequent source update) would be defined by the framework's implementation. Generally, the derivation function would re-evaluate and determine the new state.
How it differs from computed():

computed<T>(): Creates a read-only Signal<T>. Its value is solely determined by its derivation function and its dependent signals. You cannot call .set() or .update() on a computed signal.   
linkedSignal<T>(): Creates a writable WritableSignal<T>. Its value can be derived and can be set directly.
Potential Use Cases:

Form inputs with default/derived values that can be overridden: Imagine a field that defaults to a value derived from another part of the application state (e.g., "suggested price" based on other product details) but the user can also type in their own price.
Managing a "selected item" from a list: The selected item might initially be the first item of a list (derived). If the list changes, the selection might update. However, the user should also be able to click and select a different item directly, changing the linkedSignal's value.
Caching or local editable copies of state: Where a piece of data is derived, but you want to allow temporary local modifications before potentially persisting them.
Sample Code
Let's illustrate with an example of managing a selected product from a list of available products. The selected product will initially be the first available product, but the user can also "manually" select a different one.

import { Component, signal, WritableSignal, effect, computed } from '@angular/core';

// Hypothetical linkedSignal function import (official import path may vary)
// For this example, as `linkedSignal` might not be directly available in all environments
// without Angular 19 (or its specific preview), we'll demonstrate the concept.
// If Angular 19 is released and you have it, you would import it:
// import { linkedSignal } from '@angular/core';

// --- Conceptual linkedSignal (Illustrative Polyfill/Helper) ---
// This is a simplified conceptual implementation to demonstrate the idea if linkedSignal is not yet available.
// The actual Angular implementation will be more robust and integrated.
function conceptualLinkedSignal<T>(
  computation: () => T,
  sources: WritableSignal<any>[] // Simplified: list of signals that trigger re-computation
): WritableSignal<T> {
  const internalSignal = signal(computation());

  // Effect to update the internalSignal when any source changes
  effect(() => {
    internalSignal.set(computation());
  }, { allowSignalWrites: true }); // AllowSignalWrites needed if computation itself writes to signals (not typical for computation part)

  // Return a WritableSignal that allows external set/update
  // and is updated by the effect.
  // A real linkedSignal would likely be a primitive with more optimized tracking.
  return internalSignal; // In a real scenario, the returned signal would have special properties.
}
// --- End of Conceptual linkedSignal ---


interface Product {
  id: number;
  name: string;
  price: number;
}

@Component({
  selector: 'app-product-selector',
  standalone: true,
  template: `
    <div>
      <h2>Available Products:</h2>
      <ul>
        <li *ngFor="let product of availableProducts()">
          {{ product.name }} - \${{ product.price }}
          <button (click)="manuallySelectProduct(product)">Select</button>
        </li>
      </ul>
      <button (click)="loadMoreProducts()">Load More Products (Changes Source)</button>

      <hr />

      <h3>Selected Product (Managed by linkedSignal-like concept):</h3>
      <div *ngIf="selectedProduct() as product; else noProductSelected">
        <p>ID: {{ product.id }}</p>
        <p>Name: {{ product.name }}</p>
        <p>Price: \${{ product.price }}</p>
        <button (click)="clearSelection()">Clear Manual Selection (Resets to Derived)</button>
      </div>
      <ng-template #noProductSelected>
        <p>No product selected.</p>
      </ng-template>

      <p><em>Selected Product Log: {{ selectedProductLog() }}</em></p>
    </div>
  `,
  styles: [`
    hr { margin: 20px 0; }
    ul { list-style: none; padding: 0; }
    li { margin-bottom: 8px; padding: 5px; background-color: #f4f4f4; border-radius: 3px; }
  `]
})
export class ProductSelectorComponent {
  availableProducts: WritableSignal<Product[]> = signal<Product[]>([
    { id: 1, name: 'Super Keyboard', price: 75 },
    { id: 2, name: 'Ergonomic Mouse', price: 45 },
  ]);

  // Using the conceptualLinkedSignal for demonstration.
  // With official Angular 19, this would be:
  // selectedProduct: WritableSignal<Product | undefined> = linkedSignal(
  //   () => this.availableProducts()[0] // The derivation function
  //   // Angular's linkedSignal might automatically track dependencies or have a way to specify them.
  // );
  // For our conceptual version, we pass the source explicitly.
  selectedProduct: WritableSignal<Product | undefined>;

  selectedProductLog = signal('');

  constructor() {
    // Initialize selectedProduct using the conceptual helper
    // This attempts to mimic the behavior: derived but settable.
    // The `computation` function derives the value.
    // `sources` tells our conceptual helper which signals should trigger re-computation.
    const computation = () => {
      console.log('Derivation logic for selectedProduct running...');
      const products = this.availableProducts();
      return products.length > 0 ? products[0] : undefined;
    };

    // This is where the official `linkedSignal` would be used.
    // For now, using the conceptual helper:
    this.selectedProduct = signal(computation()); // Initial value

    // Effect to re-run computation if `availableProducts` changes
    effect(() => {
        const newDerivedValue = computation();
        // Only set if it's different to avoid unnecessary updates if the value is complex
        // or to allow a manual override to persist until the next actual derived change.
        // The precise interaction here is key to how `linkedSignal` would behave.
        // A simple approach for this conceptual example:
        this.selectedProduct.set(newDerivedValue);
        console.log('Source (availableProducts) changed, selectedProduct re-derived.');
    }, { allowSignalWrites: true }); // allowSignalWrites for the .set inside effect

    // Effect to log changes to the selected product
    effect(() => {
      const product = this.selectedProduct();
      const logMsg = product ? `Selected: ${product.name}` : 'Selection cleared';
      this.selectedProductLog.set(logMsg);
      console.log(logMsg);
    });
  }

  loadMoreProducts(): void {
    this.availableProducts.update(currentProducts => [
      ...currentProducts,
      { id: Date.now(), name: `New Gadget ${currentProducts.length + 1}`, price: Math.floor(Math.random() * 100) + 20 },
    ]);
    // When availableProducts changes, the effect linked to selectedProduct should
    // re-run the derivation logic, potentially updating selectedProduct to the new first item.
  }

  manuallySelectProduct(product: Product): void {
    console.log(`Manually setting selectedProduct to: ${product.name}`);
    this.selectedProduct.set(product);
    // Now selectedProduct holds the manually selected product.
    // If `loadMoreProducts` is called, the derivation logic will run again.
    // The behavior of `linkedSignal` would define if this manual set is sticky
    // or if the derivation always wins on source change.
    // Typically, the derivation would re-evaluate based on the new source.
  }

  clearSelection(): void {
    console.log('Clearing manual selection, letting derivation logic take over.');
    // To truly reset to derived, you might re-run the derivation or set to undefined
    // and let the effect pick it up. Or, if linkedSignal supports it, a specific reset method.
    // For this example, setting to undefined triggers the derivation in a way.
    // A more direct way would be to re-apply the original derivation:
    const products = this.availableProducts();
    this.selectedProduct.set(products.length > 0 ? products[0] : undefined);
  }
}


In this conceptual example:

availableProducts is the source signal.
selectedProduct is our "linkedSignal" (conceptually).
Derivation: It tries to default to the first product in availableProducts(). This derivation logic is run initially and then re-run by an effect whenever availableProducts changes.
Writable: The manuallySelectProduct() method directly calls this.selectedProduct.set(), overriding the derived value.
Behavior to Observe:
Initially, selectedProduct will be "Super Keyboard".
If you click "Load More Products", availableProducts changes. The effect monitoring it will re-run the derivation, and selectedProduct will remain "Super Keyboard" (as it's still the first). If the first item itself changed, selectedProduct would update.
If you click "Select" on "Ergonomic Mouse", selectedProduct is directly set to "Ergonomic Mouse".
If you then click "Load More Products" after a manual selection, the derivation logic for selectedProduct runs again. In our simple conceptual setup, it will again pick the first item from the updated availableProducts list, potentially overwriting your manual selection. An official linkedSignal would have well-defined semantics for how direct writes interact with source-driven updates.
Anticipated Official linkedSignal Behavior:

An official linkedSignal from Angular would likely provide a more integrated and optimized way to achieve this. The key is that the signal itself is aware of its derivation logic but also allows direct mutations, and the framework would manage the reactivity and updates efficiently. The exact way you'd define the sources and the computation, and how direct sets interact with source changes (e.g., does a direct set temporarily "win" until the next source change, or is it more complex?), would be part of its API design.

The Telerik blog post example (found in previous search results) for Angular 19 linkedSignal showed a syntax like:
selectedBook = linkedSignal(() => this.books()[0]);
or
selectedBook = linkedSignal({ source: this.books, computation: (allBooks) => allBooks[0] });

This suggests that linkedSignal internally manages the re-computation when its tracked sources change, while still providing the set() and update() methods of a WritableSignal.

