<!-- source: cf07a3f0dab8bccceb9f34d4ba811eeb800fefa4 -->
# Utiliser les signals avec les services

Maintenant que vous avez appris [le two-way binding avec les signals model](/tutorials/signals/6-two-way-binding-with-model-signals), voyons comment utiliser les signals avec les services Angular. Les services sont parfaits pour partager un état réactif entre plusieurs composants, et les signals les rendent encore plus puissants grâce à la détection de changements automatique et à des patterns réactifs clairs.

Dans cette activité, vous allez apprendre à créer un store de panier avec des signals, qui permet au composant d'affichage du panier de réagir automatiquement aux changements d'état.

<hr />

<docs-workflow>

<docs-step title="Ajouter les signals du store de panier {#add-cart-store-signals}">
Ajoutez des signals en lecture seule et des signals computed pour rendre l'état du panier réactif dans `cart-store.ts`.

```ts
// Add the computed import
import {Service, signal, computed} from '@angular/core';

// Then add these signals to the class:

// Readonly signals
readonly cartItems = this.items.asReadonly();

// Computed signals
readonly totalQuantity = computed(() => {
  return this.items().reduce((sum, item) => sum + item.quantity, 0);
});

readonly totalPrice = computed(() => {
  return this.items().reduce((sum, item) => sum + item.price * item.quantity, 0);
});
```

Ces signals permettent aux composants d'accéder de manière réactive aux données du panier et aux totaux calculés. La méthode `asReadonly()` empêche le code externe de modifier directement les articles du panier, tandis que `computed()` crée un état dérivé qui se met à jour automatiquement quand le signal source change.
</docs-step>

<docs-step title="Compléter les méthodes de mise à jour des quantités {#complete-the-quantity-update-methods}">
Le composant d'affichage du panier dans `cart-display.ts` utilise déjà les signals du store de panier dans son template. Complétez les méthodes de mise à jour des quantités pour modifier les articles du panier :

```ts
increaseQuantity(id: string) {
  const items = this.cartStore.cartItems();
  const currentItem = items.find((item) => item.id === id);
  if (currentItem) {
    this.cartStore.updateQuantity(id, currentItem.quantity + 1);
  }
}

decreaseQuantity(id: string) {
  const items = this.cartStore.cartItems();
  const currentItem = items.find((item) => item.id === id);
  if (currentItem && currentItem.quantity > 1) {
    this.cartStore.updateQuantity(id, currentItem.quantity - 1);
  }
}
```

Ces méthodes lisent l'état actuel du panier avec `cartItems()` et mettent à jour les quantités via les méthodes du store. L'interface se met à jour automatiquement quand les signals changent !
</docs-step>

<docs-step title="Mettre à jour le composant principal de l'application {#update-the-main-app-component}">
Mettez à jour le composant principal de l'application dans `app.ts` pour utiliser le service de panier et afficher le composant du panier.

```angular-ts
import {Component, inject} from '@angular/core';
import {CartStore} from './cart-store';
import {CartDisplay} from './cart-display';

@Component({
  selector: 'app-root',
  imports: [CartDisplay],
  template: `
    <div class="shopping-app">
      <header>
        <h1>Signals with Services Demo</h1>
        <div class="cart-badge">
          Cart: {{ cartStore.totalQuantity() }} items (\${{ cartStore.totalPrice() }})
        </div>
      </header>

      <main>
        <cart-display />
      </main>
    </div>
  `,
  styleUrl: './app.css',
})
export class App {
  cartStore = inject(CartStore);
}
```

</docs-step>

</docs-workflow>

Excellent ! Vous savez maintenant utiliser les signals avec les services. Concepts clés à retenir :

- **Signals au niveau des services** : les services peuvent utiliser des signals pour gérer un état réactif
- **Injection de dépendances** : utilisez `inject()` pour accéder aux services à base de signals dans les composants
- **Signals computed dans les services** : créez un état dérivé qui se met à jour automatiquement
- **Signals en lecture seule** : exposez des versions en lecture seule des signals pour empêcher les mutations externes

Dans la prochaine leçon, vous apprendrez [à utiliser les signals avec les directives](/tutorials/signals/8-using-signals-with-directives) !
