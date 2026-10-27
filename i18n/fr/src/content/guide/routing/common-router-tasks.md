<!-- source: 1d195150fcbb6fa25d24830fe827cb9b5ed4b563 -->
# Autres tâches courantes de routing

Ce guide couvre d'autres tâches courantes liées à l'utilisation du router Angular dans votre application.

## Obtenir les informations de route {#getting-route-information}

Souvent, quand un utilisateur navigue dans votre application, vous voulez transmettre des informations d'un composant à un autre.
Par exemple, imaginez une application qui affiche une liste de courses.
Chaque article de la liste possède un `id` unique.
Pour modifier un article, les utilisateurs cliquent sur un bouton Modifier, qui ouvre un composant `EditGroceryItem`.
Vous voulez que ce composant récupère l'`id` de l'article pour afficher les bonnes informations à l'utilisateur.

Utilisez une route pour transmettre ce type d'information aux composants de votre application.
Pour cela, utilisez la fonctionnalité `withComponentInputBinding` avec `provideRouter` ou l'option `bindToComponentInputs` de `RouterModule.forRoot`.

Pour obtenir des informations d'une route :

<docs-workflow>

<docs-step title="Ajouter `withComponentInputBinding` {#add-withcomponentinputbinding}">

Ajoutez la fonctionnalité `withComponentInputBinding` à la méthode `provideRouter`.

```ts
providers: [provideRouter(appRoutes, withComponentInputBinding())];
```

</docs-step>

<docs-step title="Ajouter un `input` au composant {#add-an-input-to-the-component}">

Modifiez le composant pour qu'il ait une propriété `input()` correspondant au nom du paramètre.

```ts
id = input.required<string>();
hero = computed(() => this.service.getHero(this.id()));
```

</docs-step>
<docs-step title="Facultatif : utiliser une valeur par défaut {#optional-use-a-default-value}">
Le router attribue des valeurs à tous les inputs en fonction de la route courante quand `withComponentInputBinding` est activé.
Le router attribue `undefined` si aucune donnée de route ne correspond à la clé de l'input, par exemple quand un query parameter facultatif est absent.
Incluez `undefined` dans le type de l'`input` lorsqu'il est possible qu'un input ne soit pas trouvé par la route.

Fournissez une valeur par défaut soit avec l'option `transform` de l'input, soit en gérant un état local avec un `linkedSignal`.

```ts
id = input.required({
  transform: (maybeUndefined: string | undefined) => maybeUndefined ?? '0',
});
// or
id = input<string | undefined>();
internalId = linkedSignal(() => this.id() ?? getDefaultId());
```

</docs-step>
</docs-workflow>

NOTE: Vous pouvez lier aux inputs du composant toutes les données de route sous forme de paires clé-valeur : resources de route, données de route statiques ou résolues, paramètres de chemin, paramètres matriciels et query parameters.

### Priorité du binding des inputs {#input-binding-priority}

Quand plusieurs sources de route définissent des clés identiques, le router résout les collisions selon l'ordre de priorité suivant (de la plus haute à la plus basse) :

1. **Resources de route** : les valeurs définies dans la map `resources` de la route. Les resources bloquantes lient leur valeur déballée (`resource.value()`), tandis que les resources non bloquantes lient l'instance de `Resource`.
2. **Données de route et resolvers** : les données statiques définies dans `data` ou les valeurs résolues via `resolve`.
3. **Paramètres de chemin et paramètres matriciels** : les paramètres issus du chemin de l'URL (comme `:id`) et les paramètres matriciels.
4. **Query parameters** : les paramètres issus de la query string (comme `?id=123`).

Par exemple, si une route possède à la fois un paramètre de chemin `:id` et un query parameter `?id=...`, la valeur du paramètre de chemin est liée à l'input `id` du composant. Si la route définit aussi une resource nommée `id`, la valeur de la resource l'emporte sur les deux.

### Désactiver le binding des query parameters {#disable-query-parameter-binding}

Utilisez `ComponentInputBindingOptions` pour désactiver le binding des query parameters si vous gérez les query parameters séparément :

```ts
provideRouter(appRoutes, withComponentInputBinding({queryParams: false}));
```

### Configurer le comportement des inputs absents des données du router {#configure-behavior-for-inputs-not-available-in-router-data}

Par défaut, le router définit un input à `undefined` s'il n'était pas disponible dans les données du router lors d'une navigation. Cela garantit que des données obsolètes ne sont pas conservées.

Si vous voulez éviter de définir `undefined` pour les inputs qui n'ont _jamais_ été disponibles dans les données du router pour l'instance de composant active, vous pouvez définir l'option `unmatchedInputBehavior` à `'undefinedIfStale'` :

```ts
provideRouter(appRoutes, withComponentInputBinding({unmatchedInputBehavior: 'undefinedIfStale'}));
```

Quand vous combinez `unmatchedInputBehavior: 'undefinedIfStale'` avec `queryParams: false`, les inputs conservent leurs valeurs initiales sauf s'ils sont explicitement fournis par le router. L'exception concerne les paramètres matriciels : si un paramètre matriciel est fourni lors d'une navigation puis retiré lors d'une suivante, le router définit l'input à `undefined` pour éviter de conserver des données obsolètes.

```ts
provideRouter(
  appRoutes,
  withComponentInputBinding({
    queryParams: false,
    unmatchedInputBehavior: 'undefinedIfStale',
  }),
);
```

### Hériter des données de la route parente {#inherit-parent-route-data}

Par défaut, les routes enfants héritent des paramètres et des données des routes parentes (équivalent à `paramsInheritanceStrategy: 'always'`). Vous pouvez donc accéder directement aux informations de la route parente dans les composants enfants.

Si vous devez restaurer l'ancien comportement, où les paramètres n'étaient hérités que des routes à chemin vide, vous pouvez définir `paramsInheritanceStrategy` à `'emptyOnly'` :

```ts
provideRouter(routes, withRouterConfig({paramsInheritanceStrategy: 'emptyOnly'}));
```

Consultez les [options de configuration du router](guide/routing/customizing-route-behavior#router-configuration-options) pour les autres paramètres disponibles.

## Afficher une page 404 {#displaying-a-404-page}

Pour afficher une page 404, configurez une [route wildcard](guide/routing/define-routes#wildcards) dont la propriété `component` désigne le composant à utiliser pour votre page 404, comme ceci :

```ts
const routes: Routes = [
  {path: 'first-component', component: First},
  {path: 'second-component', component: Second},
  {path: '**', component: PageNotFound}, // Wildcard route for a 404 page
];
```

La dernière route, dont le `path` vaut `**`, est une route wildcard.
Le router sélectionne cette route si l'URL demandée ne correspond à aucun des chemins précédents de la liste et envoie l'utilisateur vers `PageNotFound`.

## Tableau de paramètres de lien {#link-parameters-array}

Un tableau de paramètres de lien contient les éléments suivants pour la navigation du router :

- Le chemin de la route vers le composant de destination
- Les paramètres de route obligatoires et facultatifs qui entrent dans l'URL de la route

Liez la directive `RouterLink` à un tel tableau comme ceci :

```angular-html
<a [routerLink]="['/heroes']">Heroes</a>
```

Voici un tableau à deux éléments lorsqu'on spécifie un paramètre de route :

```angular-html
<a [routerLink]="['/hero', hero.id]">
  <span class="badge">{{ hero.id }}</span
  >{{ hero.name }}
</a>
```

Fournissez les paramètres de route facultatifs dans un objet, comme dans `{ foo: 'foo' }` :

```angular-html
<a [routerLink]="['/crisis-center', {foo: 'foo'}]">Crisis Center</a>
```

Cette syntaxe transmet des paramètres matriciels, qui sont des paramètres facultatifs associés à un segment d'URL précis. En savoir plus sur les [paramètres matriciels](/guide/routing/read-route-state#matrix-parameters).

Ces trois exemples couvrent les besoins d'une application avec un seul niveau de routing.
Cependant, avec un router enfant, comme dans le centre de crise, vous créez de nouvelles possibilités de tableaux de liens.

L'exemple minimal de `RouterLink` suivant s'appuie sur une route enfant par défaut spécifiée pour le centre de crise.

```angular-html
<a [routerLink]="['/crisis-center']">Crisis Center</a>
```

Remarquez les points suivants :

- Le premier élément du tableau identifie la route parente \(`/crisis-center`\)
- Cette route parente n'a pas de paramètres
- Il n'y a pas de valeur par défaut pour la route enfant, vous devez donc en choisir une
- Vous naviguez vers `CrisisList`, dont le chemin de route est `/`, mais vous n'avez pas besoin d'ajouter explicitement la barre oblique

Considérez le lien de router suivant, qui navigue de la racine de l'application jusqu'à la Dragon Crisis :

```angular-html
<a [routerLink]="['/crisis-center', 1]">Dragon Crisis</a>
```

- Le premier élément du tableau identifie la route parente \(`/crisis-center`\)
- Cette route parente n'a pas de paramètres
- Le deuxième élément identifie la route enfant de détail d'une crise particulière \(`/:id`\)
- La route enfant de détail nécessite un paramètre de route `id`
- Vous avez ajouté l'`id` de la Dragon Crisis comme deuxième élément du tableau \(`1`\)
- Le chemin obtenu est `/crisis-center/1`

Vous pourriez aussi redéfinir le template de `App` exclusivement avec les routes du Crisis Center :

```angular-ts
@Component({
  template: `
    <h1 class="title">Angular Router</h1>
    <nav>
      <a [routerLink]="['/crisis-center']">Crisis Center</a>
      <a [routerLink]="['/crisis-center/1', {foo: 'foo'}]">Dragon Crisis</a>
      <a [routerLink]="['/crisis-center/2']">Shark Crisis</a>
    </nav>
    <router-outlet />
  `,
})
export class App {}
```

En résumé, vous pouvez écrire des applications avec un, deux niveaux de routing ou plus.
Le tableau de paramètres de lien offre la flexibilité nécessaire pour représenter n'importe quelle profondeur de routing et n'importe quelle séquence valide de chemins de route, de paramètres de router \(obligatoires\) et d'objets de paramètres de route \(facultatifs\).

## `LocationStrategy` et styles d'URL du navigateur {#locationstrategy-and-browser-url-styles}

Quand le router navigue vers une nouvelle vue de composant, il met à jour l'emplacement et l'historique du navigateur avec une URL pour cette vue.

Les navigateurs HTML5 modernes prennent en charge [history.pushState](https://developer.mozilla.org/docs/Web/API/History_API/Working_with_the_History_API#adding_and_modifying_history_entries 'HTML5 browser history push-state'), une technique qui modifie l'emplacement et l'historique du navigateur sans déclencher de requête de page vers le serveur.
Le router peut composer une URL « naturelle », impossible à distinguer d'une URL qui nécessiterait autrement un chargement de page.

Voici l'URL du Crisis Center dans ce style « HTML5 pushState » :

```text
localhost:3002/crisis-center
```

Les navigateurs plus anciens envoient des requêtes de page au serveur quand l'URL de l'emplacement change, sauf si le changement se produit après un « # » \(appelé « hash »\).
Les routers peuvent tirer parti de cette exception en composant des URL de route internes à l'application avec des hashes.
Voici une « URL hash » qui mène au Crisis Center.

```text
localhost:3002/src/#/crisis-center
```

Le router prend en charge les deux styles avec deux providers de `LocationStrategy` :

| Providers              | Détails                              |
| :--------------------- | :----------------------------------- |
| `PathLocationStrategy` | Le style « HTML5 pushState » par défaut. |
| `HashLocationStrategy` | Le style « URL hash ».                |

La fonction `RouterModule.forRoot()` définit la `LocationStrategy` sur `PathLocationStrategy`, ce qui en fait la stratégie par défaut.
Vous pouvez aussi passer à `HashLocationStrategy` avec une surcharge pendant le processus de bootstrap.

HELPFUL: Pour plus d'informations sur les providers et le processus de bootstrap, consultez [Injection de dépendances](guide/di/defining-dependency-providers).
