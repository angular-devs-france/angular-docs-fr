<!-- source: dd4888be06cb79d7995e1835e86d61a191d92b40 -->
# Référence du router

Les sections suivantes présentent quelques concepts et termes essentiels du router.

## Événements du router {#router-events}

Pendant chaque navigation, le `Router` émet des événements de navigation via la propriété `Router.events`.
Ces événements sont présentés dans le tableau suivant.

| Événement du router                                       | Détails                                                                                                                                                                |
| :-------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`NavigationStart`](api/router/NavigationStart)           | Déclenché au début de la navigation. |
| [`RouteConfigLoadStart`](api/router/RouteConfigLoadStart) | Déclenché avant que le `Router` charge une configuration de route en lazy loading. |
| [`RouteConfigLoadEnd`](api/router/RouteConfigLoadEnd)     | Déclenché après le chargement d'une route en lazy loading. |
| [`RoutesRecognized`](api/router/RoutesRecognized)         | Déclenché quand le router analyse l'URL et que les routes sont reconnues. |
| [`GuardsCheckStart`](api/router/GuardsCheckStart)         | Déclenché quand le router commence la phase des guards du routing. |
| [`ChildActivationStart`](api/router/ChildActivationStart) | Déclenché quand le router commence à activer les enfants d'une route. |
| [`ActivationStart`](api/router/ActivationStart)           | Déclenché quand le router commence à activer une route. |
| [`GuardsCheckEnd`](api/router/GuardsCheckEnd)             | Déclenché quand le router termine avec succès la phase des guards du routing. |
| [`ResolveStart`](api/router/ResolveStart)                 | Déclenché quand le router commence la phase de résolution (Resolve) du routing. |
| [`ResolveEnd`](api/router/ResolveEnd)                     | Déclenché quand le router termine avec succès la phase de résolution (Resolve) du routing. |
| [`ChildActivationEnd`](api/router/ChildActivationEnd)     | Déclenché quand le router termine l'activation des enfants d'une route. |
| [`ActivationEnd`](api/router/ActivationEnd)               | Déclenché quand le router termine l'activation d'une route. |
| [`NavigationEnd`](api/router/NavigationEnd)               | Déclenché quand la navigation se termine avec succès. |
| [`NavigationCancel`](api/router/NavigationCancel)         | Déclenché quand la navigation est annulée. Cela peut se produire quand une guard de route renvoie false pendant la navigation, ou effectue une redirection en renvoyant un `UrlTree` ou une `RedirectCommand`. |
| [`NavigationError`](api/router/NavigationError)           | Déclenché quand la navigation échoue à cause d'une erreur inattendue. |
| [`Scroll`](api/router/Scroll)                             | Représente un événement de défilement. |

Quand vous activez la fonctionnalité `withDebugTracing`, Angular affiche ces événements dans la console.

## Terminologie du router {#router-terminology}

Voici les principaux termes du `Router` et leur signification :

| Élément du router     | Détails                                                                                                                                                                                                                                   |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Router`              | Affiche le composant de l'application correspondant à l'URL active. Gère la navigation d'un composant à l'autre. |
| `provideRouter`       | Fournit les providers de services nécessaires pour naviguer entre les vues de l'application. |
| `RouterModule`        | Un NgModule distinct qui fournit les providers de services et les directives nécessaires pour naviguer entre les vues de l'application. |
| `Routes`              | Définit un tableau de routes, chacune associant un chemin d'URL à un composant. |
| `Route`               | Définit comment le router doit naviguer vers un composant en fonction d'un motif d'URL. Les routes peuvent inclure des chemins, des composants, des routes enfants, des guards, des resolvers et des resources. |
| `RouterOutlet`        | La directive \(`<router-outlet>`\) qui indique où le router affiche une vue. |
| `RouterLink`          | La directive qui crée un binding entre un élément HTML cliquable et une route. Cliquer sur un élément portant une directive `routerLink` liée à une _chaîne_ ou à un _tableau de paramètres de lien_ déclenche une navigation. |
| `RouterLinkActive`    | La directive qui ajoute ou retire des classes sur un élément HTML quand un `routerLink` associé, placé sur l'élément ou à l'intérieur, devient actif ou inactif. Elle peut aussi définir l'attribut `aria-current` d'un lien actif pour une meilleure accessibilité. |
| `ActivatedRoute`      | Un service fourni à chaque composant de route, qui contient des informations propres à la route comme les paramètres de route, les données statiques, les données résolues, les resources de la route, les query parameters globaux et le fragment global. |
| `RouterState`         | L'état courant du router, qui comprend un arbre des routes actuellement activées ainsi que des méthodes pratiques pour parcourir l'arbre de routes. |
| Tableau de paramètres de lien | Un tableau que le router interprète comme une instruction de routing. Vous pouvez lier ce tableau à un `RouterLink` ou le passer en argument à la méthode `Router.navigate`. |
| Composant de routing  | Un composant Angular doté d'un `RouterOutlet`, qui affiche des vues en fonction des navigations du router. |

## `<base href>`

Le router utilise la méthode [history.pushState](https://developer.mozilla.org/docs/Web/API/History_API/Working_with_the_History_API#adding_and_modifying_history_entries 'HTML5 browser history push-state') du navigateur pour la navigation.
`pushState` vous permet de personnaliser les chemins d'URL internes à l'application ; par exemple, `localhost:4200/crisis-center`.
Les URL internes à l'application peuvent être impossibles à distinguer des URL du serveur.

Les navigateurs HTML5 modernes ont été les premiers à prendre en charge `pushState`, c'est pourquoi beaucoup de gens appellent ces URL des URL « de style HTML5 ».

HELPFUL: La navigation de style HTML5 est le comportement par défaut du router.
Dans la section [LocationStrategy et styles d'URL du navigateur](guide/routing/common-router-tasks#locationstrategy-and-browser-url-styles), découvrez pourquoi le style HTML5 est préférable, comment ajuster son comportement et comment passer à l'ancien style avec hash \(`#`\) si nécessaire.

Vous devez ajouter un [élément `<base href>`](https://developer.mozilla.org/docs/Web/HTML/Element/base 'base href') au fichier `index.html` de l'application pour que le routing par `pushState` fonctionne.
Le navigateur utilise la valeur de `<base href>` pour préfixer les URL relatives lorsqu'il référence des fichiers CSS, des scripts et des images.

Ajoutez l'élément `<base>` juste après la balise `<head>`.
Si le dossier `app` est la racine de l'application, comme c'est le cas pour cette application, définissez la valeur `href` dans `index.html` comme indiqué ici.

```html
<base href="/" />
```

### Les URL HTML5 et le `<base href>` {#html5-urls-and-the-base-href}

Les recommandations qui suivent font référence à différentes parties d'une URL.
Ce diagramme indique à quoi correspondent ces parties :

```text {hideCopy}
foo://example.com:8042/over/there?name=ferret#nose
\_/   \______________/\_________/ \_________/ \__/
 |           |            |            |        |
scheme    authority      path        query   fragment
```

Bien que le router utilise par défaut le style [HTML5 pushState](https://developer.mozilla.org/docs/Web/API/History_API#Adding_and_modifying_history_entries 'Browser history push-state'), vous devez configurer cette stratégie avec un `<base href>`.

La manière recommandée de configurer la stratégie consiste à ajouter une balise [élément `<base href>`](https://developer.mozilla.org/docs/Web/HTML/Element/base 'base href') dans le `<head>` du fichier `index.html`.

```angular-html
<base href="/" />
```

Sans cette balise, le navigateur risque de ne pas pouvoir charger les ressources \(images, CSS, scripts\) lors d'un accès par « lien profond » (deep linking) dans l'application.

Certains développeurs ne peuvent pas ajouter l'élément `<base>`, par exemple parce qu'ils n'ont pas accès au `<head>` ou au fichier `index.html`.

Ces développeurs peuvent tout de même utiliser des URL HTML5 en suivant les deux étapes suivantes :

1. Fournissez au router une valeur `APP_BASE_HREF` appropriée.
1. Utilisez des URL racines \(URL avec une `authority`\) pour toutes les ressources web : CSS, images, scripts et fichiers HTML de template.
   - Le `path` du `<base href>` doit se terminer par un « / », car les navigateurs ignorent les caractères du `path` qui suivent le « `/` » le plus à droite
   - Si le `<base href>` contient une partie `query`, cette `query` n'est utilisée que si le `path` d'un lien de la page est vide et n'a pas de `query`.
     Cela signifie qu'une `query` dans le `<base href>` n'est incluse que lorsque vous utilisez `HashLocationStrategy`.

   - Si un lien de la page est une URL racine \(il a une `authority`\), le `<base href>` n'est pas utilisé.
     Ainsi, un `APP_BASE_HREF` avec une authority fait que tous les liens créés par Angular ignorent la valeur de `<base href>`.

   - Un fragment dans le `<base href>` n'est _jamais_ conservé

Pour des informations plus complètes sur la façon dont `<base href>` est utilisé pour construire les URI cibles, consultez la section de la [RFC](https://tools.ietf.org/html/rfc3986#section-5.2.2) consacrée à la transformation des références.

### `HashLocationStrategy`

Utilisez `HashLocationStrategy` en passant `useHash: true` dans un objet en second argument de `RouterModule.forRoot()` dans l'`AppModule`.

```ts
providers: [provideRouter(appRoutes, withHashLocation())];
```

Avec `RouterModule.forRoot`, cela se configure avec `useHash: true` dans le second argument : `RouterModule.forRoot(routes, {useHash: true})`.
