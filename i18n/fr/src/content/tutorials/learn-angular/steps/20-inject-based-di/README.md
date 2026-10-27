<!-- source: b16fd1d24a276d85c3976263f9f59d5631c8dcdd -->
# Injection de dépendances avec inject

Créer un service injectable constitue la première partie du système d'injection de dépendances (DI) d'Angular. Comment injecter un service dans un composant ? Angular fournit une fonction pratique appelée `inject()`, utilisable dans le contexte approprié.

NOTE: Les contextes d'injection dépassent le cadre de ce tutoriel, mais vous pouvez en apprendre davantage dans le [guide des Fondamentaux sur l'injection de dépendances (DI)](/essentials/dependency-injection) et dans le [guide sur le contexte d'injection](guide/di/dependency-injection-context).

Dans cette activité, vous allez apprendre à injecter un service et à l'utiliser dans un composant.

<hr>

Il est souvent utile d'initialiser des propriétés de classe avec des valeurs fournies par le système de DI. Voici un exemple :

```ts {highlight:[3]}
@Component({...})
class PetCareDashboard {
  petRosterService = inject(PetRosterService);
}
```

<docs-workflow>

<docs-step title="Injecter le `CarService` {#inject-the-carservice}">

Dans `app.ts`, injectez le `CarService` à l'aide de la fonction `inject()` et assignez-le à une propriété appelée `carService`.

NOTE: Remarquez la différence entre la propriété `carService` et la classe `CarService`.

</docs-step>

<docs-step title="Utiliser l'instance `carService` {#use-the-carservice-instance}">

L'appel à `inject(CarService)` vous a fourni une instance de `CarService` utilisable dans votre application, stockée dans la propriété `carService`.

Initialisez la propriété `display` avec l'implémentation suivante :

```ts
display = this.carService.getCars().join(' ⭐️ ');
```

</docs-step>

<docs-step title="Mettre à jour le template de `App` {#update-the-app-template}">

Mettez à jour le template du composant dans `app.ts` avec le code suivant :

```ts
template: `<p>Car Listing: {{ display }}</p>`,
```

</docs-step>

</docs-workflow>

Vous venez d'injecter votre premier service dans un composant : excellent travail.
