<!-- source: 8354746f5de3140309f14cdaa3ec3233693d492c -->
# Accéder aux éléments enfants avec des queries signal

Maintenant que vous avez appris [à utiliser les signals avec les directives](/tutorials/signals/8-using-signals-with-directives), découvrons les API de query basées sur les signals. Elles offrent un moyen réactif d'accéder aux composants et directives enfants et d'interagir avec eux. Les composants comme les directives peuvent effectuer des queries tout en étant eux-mêmes la cible de queries. Contrairement au ViewChild traditionnel, les queries signal se mettent à jour automatiquement et offrent un accès typé aux composants et directives enfants.

Dans cette activité, vous allez ajouter des queries viewChild pour interagir avec des composants enfants par programmation.

<hr />

<docs-workflow>

<docs-step title="Ajouter l'import de viewChild {#add-viewchild-import}">
Commencez par ajouter l'import de `viewChild` pour accéder aux composants enfants dans `app.ts`.

```ts
import {Component, signal, computed, viewChild, ChangeDetectionStrategy} from '@angular/core';
```

</docs-step>

<docs-step title="Créer des queries viewChild {#create-viewchild-queries}">
Ajoutez des queries viewChild au composant App pour accéder aux composants enfants.

```ts
// Query APIs to access child components
firstProduct = viewChild(ProductCard);
cartSummary = viewChild(CartSummary);
```

Ces queries créent des signals qui référencent les instances des composants enfants.
</docs-step>

<docs-step title="Implémenter les méthodes du parent {#implement-parent-methods}">
Utilisez les queries viewChild pour appeler des méthodes sur les composants enfants dans `app.ts` :

```ts
showFirstProductDetails() {
  const product = this.firstProduct();
  if (product) {
    product.highlight();
  }
}

initiateCheckout() {
  const summary = this.cartSummary();
  if (summary) {
    summary.initiateCheckout();
  }
}
```

</docs-step>

<docs-step title="Tester les interactions {#test-the-interactions}">
Les boutons de contrôle devraient maintenant fonctionner :

- **« Show First Product Details »** : appelle `highlight()` sur le ProductCard
- **« Initiate Checkout »** : appelle `initiateCheckout()` sur le CartSummary

Cliquez sur les boutons pour voir comment les queries viewChild permettent aux composants parents de contrôler le comportement des enfants.
</docs-step>

</docs-workflow>

Parfait ! Vous avez appris à utiliser les API de query basées sur les signals pour interagir avec des composants enfants :

Dans la prochaine leçon, vous apprendrez [à réagir aux changements de signals avec effect](/tutorials/signals/10-reacting-to-signal-changes-with-effect) !
