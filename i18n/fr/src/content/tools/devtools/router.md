<!-- source: 6edaf864baf13ef9f4425be2ff153124356b82f0 -->
# Inspecter l'arbre du router

L'onglet **Router Tree** vous permet de visualiser l'arbre de routing de votre application. Vous pouvez explorer l'imbrication des routes et afficher les détails de routes spécifiques.

<img src="assets/images/guide/devtools/router-tree.png" alt="Une capture d'écran de l'onglet 'Router Tree' d'Angular DevTools montrant un arbre de routes configurées. Les routes actives sont mises en évidence en vert, les routes inactives sont en blanc.">

### Afficher les détails d'une route {#view-route-details}

Lorsque vous sélectionnez une route dans l'arbre, Angular DevTools affiche ses propriétés dans la barre latérale de droite. Ces informations comprennent :

- **Path** : le chemin d'URL de la route. Si la route utilise un URL matcher personnalisé, DevTools affiche à la place le **Matcher**.
- **Component** : le composant affiché pour cette route. Si la route est une redirection, DevTools affiche à la place la cible **Redirect to**.
- **Path Match** : la stratégie de correspondance du chemin (`prefix` ou `full`), si elle est configurée.
- **Data** : les données statiques associées à la route, affichées sous forme d'arbre JSON.
- **Resolvers** : les resolvers de la route, affichés sous forme de paires clé-valeur.
- **Guards** : les guards configurées sur la route, regroupées par type : `canActivate`, `canActivateChild`, `canDeactivate` et `canMatch`.
- **Providers** : les providers au niveau de la route, s'ils sont configurés.
- **Title** : le titre de la route, s'il est configuré.
- **RunGuardsAndResolvers** : la stratégie de réexécution des guards et des resolvers, si elle est configurée.
- **Active** : indique si cette route est actuellement active.
- **Auxiliary** : indique si la route est une route auxiliaire (par exemple, dans un outlet nommé).
- **Lazy** : indique si la route est chargée en lazy loading.

NOTE: Les propriétés comme Path Match, Data, Resolvers, Guards, Providers, Title et RunGuardsAndResolvers n'apparaissent dans la barre latérale que lorsqu'elles sont configurées sur la route sélectionnée.

### Naviguer vers une route spécifique {#navigate-to-a-specific-route}

Vous pouvez facilement déclencher une navigation directement depuis les DevTools. Pendant que vous consultez les détails d'une route dans la barre latérale de droite, cliquez sur l'icône **Navigate** à côté de la chaîne du chemin. Le router d'Angular navigue alors vers cette URL dans votre application.

<img src="assets/images/guide/devtools/router-tree-navigate.png" alt="Une capture d'écran montrant l'infobulle 'Navigate to' sur le chemin de la route dans la barre latérale 'Routes Details'.">
