<!-- source: b859f8d03628d316b4cc7f462cf882f811350ed7 -->
# Passer des données aux composants avec les inputs signal

Maintenant que vous avez appris [à gérer des données asynchrones avec des signals](/tutorials/signals/4-managing-async-data-with-signals), découvrons l'API `input()` d'Angular, basée sur les signals, qui permet de passer des données d'un composant parent à un composant enfant et rend le flux de données entre composants plus réactif et plus efficace. Si vous connaissez les props de composant d'autres frameworks, les inputs reposent sur la même idée.

Dans cette activité, vous allez ajouter des inputs signal à un composant de carte produit et observer comment les données du parent descendent de manière réactive.

<hr />

<docs-workflow>

<docs-step title="Ajouter des inputs signal à ProductCard {#add-signal-inputs-to-productcard}">
Ajoutez des fonctions signal `input()` pour recevoir des données dans le composant `product-card`.

```ts
// Add imports for signal inputs
import {Component, input, ChangeDetectionStrategy} from '@angular/core';

// Add these signal inputs
name = input.required<string>();
price = input.required<number>();
available = input<boolean>(true);
```

Remarquez que `input.required()` crée un input qui doit obligatoirement être fourni, alors que `input()` avec une valeur par défaut est facultatif.
</docs-step>

<docs-step title="Connecter les inputs au template {#connect-inputs-to-the-template}">
Mettez à jour le template de `product-card` pour afficher les valeurs des inputs signal.

```angular-html
<div class="product-card">
  <h3>{{ name() }}</h3>
  <p class="price">\${{ price() }}</p>
  <p class="status">
    Status:
    @if (available()) {
      Available
    } @else {
      Out of Stock
    }
  </p>
</div>
```

Les inputs signal fonctionnent exactement comme les signals classiques dans les templates : appelez-les comme des fonctions pour accéder à leurs valeurs.
</docs-step>

<docs-step title="Connecter les signals du parent aux inputs de l'enfant {#connect-parent-signals-to-child-inputs}">
Mettez à jour l'utilisation de `product-card` dans `app.ts` pour passer des valeurs de signals dynamiques au lieu de valeurs statiques.

```html
<!-- Change from static values: -->
<product-card name="Static Product" price="99" available="true" />

<!-- To dynamic signals: -->
<product-card [name]="productName()" [price]="productPrice()" [available]="productAvailable()" />
```

Les crochets `[]` créent des property bindings qui passent les valeurs actuelles des signals à l'enfant.
</docs-step>

<docs-step title="Tester les mises à jour réactives {#test-reactive-updates}">
Ajoutez dans `app.ts` des méthodes pour mettre à jour les signals du parent et observez comment le composant enfant réagit automatiquement.

```ts
updateProduct() {
  this.productName.set(`Product ${Math.floor(Math.random() * 100)}`);
  this.productPrice.set(Math.floor(Math.random() * 500) + 50);
}

toggleAvailability() {
  this.productAvailable.set(!this.productAvailable());
}
```

```html
<!-- Add controls to test reactivity -->
<div class="controls">
  <button (click)="updateProduct()">Update Product Data</button>
  <button (click)="toggleAvailability()">Toggle Availability</button>
</div>
```

Quand les signals du parent changent, le composant enfant reçoit et affiche automatiquement les nouvelles valeurs !
</docs-step>

</docs-workflow>

Excellent ! Vous avez appris comment fonctionnent les inputs signal :

- **Inputs signal** : utilisez `input()` et `input.required()` pour recevoir des données des composants parents
- **Mises à jour réactives** : les composants enfants se mettent à jour automatiquement quand les valeurs des signals du parent changent
- **Sûreté du typage** : les inputs signal bénéficient de la vérification de types complète de TypeScript
- **Valeurs par défaut** : les inputs facultatifs peuvent avoir des valeurs par défaut, alors que les inputs obligatoires doivent être fournis

Les inputs signal rendent la communication entre composants plus réactive et, dans bien des cas, éliminent le besoin du lifecycle hook `OnChanges`.

Dans la prochaine leçon, vous découvrirez [le two-way binding avec les signals model](/tutorials/signals/6-two-way-binding-with-model-signals) !
