<!-- source: faa9ced6fe703368592d2a6d84921988a53de148 -->
# Gérer des données asynchrones avec des signals grâce à l'API Resource

Maintenant que vous avez appris [à dériver un état avec des linked signals](/tutorials/signals/3-deriving-state-with-linked-signals), voyons comment gérer des données asynchrones avec l'API Resource. L'API Resource offre un moyen puissant de gérer des opérations asynchrones avec des signals, avec des états de chargement, une gestion des erreurs et une gestion des requêtes intégrés.

Dans cette activité, vous allez apprendre à utiliser la fonction `resource()` pour charger des données de manière asynchrone et à gérer les différents états des opérations asynchrones, en construisant un loader de profil utilisateur qui montre l'API Resource en action.

<hr />

<docs-workflow>

<docs-step title="Importer la fonction resource et l'API {#import-resource-function-and-api}">
Ajoutez `resource` à vos imports existants et importez la fonction de l'API simulée.

```ts
// Add resource to existing imports
import {Component, signal, computed, resource, ChangeDetectionStrategy} from '@angular/core';
// Import mock API function
import {getUserData} from './user-api';
```

</docs-step>

<docs-step title="Créer une resource pour les données utilisateur {#create-a-resource-for-user-data}">
Ajoutez dans la classe du composant une propriété qui crée une resource pour charger les données utilisateur à partir d'un signal d'identifiant utilisateur.

```ts
userId = signal(1);

userResource = resource({
  params: () => ({id: this.userId()}),
  loader: (params) => getUserData(params.params.id),
});
```

</docs-step>

<docs-step title="Ajouter des méthodes pour interagir avec la resource {#add-methods-to-interact-with-the-resource}">
Ajoutez des méthodes pour changer l'identifiant utilisateur et recharger la resource.

```ts
loadUser(id: number) {
  this.userId.set(id);
}

reloadUser() {
  this.userResource.reload();
}
```

Changer le signal params déclenche automatiquement un rechargement ; vous pouvez aussi recharger manuellement avec `reload()`.
</docs-step>

<docs-step title="Créer des signals computed pour les états de la resource {#create-computed-signals-for-resource-states}">
Ajoutez des signals computed pour accéder aux différents états de la resource.

```ts
isLoading = this.userResource.isLoading;
hasError = computed(() => this.userResource.status() === 'error');
```

Les resources fournissent un signal `status()` qui peut valoir 'loading', 'reloading', 'resolved' ou 'error', un signal `value()` pour les données chargées, et une méthode `hasValue()` qui vérifie de manière sûre si des données sont disponibles. Le signal `isLoading()` vaut true pour 'loading' comme pour 'reloading'.
</docs-step>

<docs-step title="Brancher les boutons et afficher les états de la resource {#wire-up-the-buttons-and-display-resource-states}">
La structure du template est déjà fournie. Connectez maintenant l'ensemble :

Partie 1. **Ajoutez des gestionnaires de clic aux boutons :**

```html
<button (click)="loadUser(1)">Load User 1</button>
<button (click)="loadUser(2)">Load User 2</button>
<button (click)="loadUser(999)">Load Invalid User</button>
<button (click)="reloadUser()">Reload</button>
```

Partie 2. **Remplacez le placeholder par la gestion des états de la resource :**

```angular-html
@if (isLoading()) {
  <p>Loading user...</p>
} @else if (hasError()) {
  <p class="error">Error: {{ userResource.error()?.message }}</p>
} @else if (userResource.hasValue()) {
  <div class="user-info">
    <h3>{{ userResource.value().name }}</h3>
    <p>{{ userResource.value().email }}</p>
  </div>
}
```

La resource fournit différentes méthodes pour vérifier son état :

- `isLoading()` : true pendant la récupération des données
- `hasError()` : true quand une erreur s'est produite
- `userResource.hasValue()` : true quand des données sont disponibles
- `userResource.value()` : accède aux données chargées
- `userResource.error()` : accède aux informations sur l'erreur

</docs-step>

</docs-workflow>

Excellent ! Vous savez maintenant utiliser l'API Resource avec les signals. Concepts clés à retenir :

- **Les resources sont réactives** : elles se rechargent automatiquement quand les params changent
- **Gestion d'état intégrée** : les resources fournissent les signals `status()`, `value()` et `error()`
- **Nettoyage automatique** : les resources gèrent automatiquement l'annulation et le nettoyage des requêtes
- **Contrôle manuel** : vous pouvez recharger ou annuler manuellement les requêtes si nécessaire

Dans la prochaine leçon, vous apprendrez [à passer des données aux composants avec les inputs signal](/tutorials/signals/5-component-communication-with-signals) !
