<!-- source: 281791019e74f23ef3c6451d898a0b90a6b91feb -->
# Inspecter l'arbre de composants

## Déboguer votre application {#debug-your-application}

L'onglet **Components** vous permet d'explorer la structure de votre application.
Vous pouvez visualiser les instances de composants et de directives dans le DOM, et inspecter ou modifier leur état.

### Explorer la structure de l'application {#explore-the-application-structure}

L'arbre de composants affiche la relation hiérarchique entre les _composants et directives_ de votre application.

<img src="assets/images/guide/devtools/component-explorer.png" alt="Une capture d'écran de l'onglet 'Components' montrant un arbre de composants et de directives Angular à partir de la racine de l'application.">

Cliquez sur les composants ou directives dans l'explorateur de composants pour les sélectionner et prévisualiser leurs propriétés.
Angular DevTools affiche les propriétés et les métadonnées à droite de l'arbre de composants.

Pour rechercher un composant ou une directive par son nom, utilisez le champ de recherche au-dessus de l'arbre de composants.

<img src="assets/images/guide/devtools/search.png" alt="Une capture d'écran de l'onglet 'Components'. La barre de filtre située juste sous l'onglet recherche 'todo', et tous les composants dont le nom contient 'todo' sont mis en évidence dans l'arbre. `app-todos` est actuellement sélectionné et une barre latérale à droite affiche des informations sur les propriétés du composant, dont une section pour les champs `@Output` et une autre pour les autres propriétés.">

### Accéder au nœud host {#navigate-to-the-host-node}

Pour accéder au host element d'un composant ou d'une directive, double-cliquez dessus dans l'explorateur de composants.
Angular DevTools ouvre l'onglet Elements dans Chrome ou l'onglet Inspector dans Firefox, et sélectionne le nœud DOM associé.

### Accéder au code source {#navigate-to-source}

Pour les composants, Angular DevTools vous permet d'accéder à la définition du composant dans l'onglet Sources (Chrome) et l'onglet Debugger (Firefox).
Après avoir sélectionné un composant, cliquez sur l'icône en haut à droite de la vue des propriétés :

<img src="assets/images/guide/devtools/navigate-source.png" alt="Une capture d'écran de l'onglet 'Components'. La vue des propriétés d'un composant est visible à droite et la souris est placée dans le coin supérieur droit de cette vue, sur une icône `<>`. Une infobulle adjacente indique 'Open component source'.">

### Modifier la valeur d'une propriété {#update-property-value}

Comme dans les DevTools des navigateurs, la vue des propriétés vous permet de modifier la valeur d'un input, d'un output ou d'autres propriétés.
Faites un clic droit sur la valeur de la propriété : si la modification est disponible pour ce type de valeur, un champ de saisie apparaît.
Saisissez la nouvelle valeur et appuyez sur `Enter` pour l'appliquer à la propriété.

<img src="assets/images/guide/devtools/update-property.png" alt="Une capture d'écran de l'onglet 'Components' avec la vue des propriétés ouverte pour un composant. Un `@Input` nommé `todo` contient une propriété `label`, actuellement sélectionnée, dont la valeur a été modifiée manuellement en 'Buy milk'.">

### Accéder au composant ou à la directive sélectionné dans la console {#access-selected-component-or-directive-in-console}

En guise de raccourci dans la console, Angular DevTools donne accès aux instances des composants ou directives récemment sélectionnés.
Saisissez `$ng0` pour obtenir une référence à l'instance du composant ou de la directive actuellement sélectionné, `$ng1` pour l'instance sélectionnée précédemment, `$ng2` pour celle sélectionnée avant, et ainsi de suite.

<img src="assets/images/guide/devtools/access-console.png" alt="Une capture d'écran de l'onglet 'Components' avec la console du navigateur en dessous. Dans la console, l'utilisateur a saisi trois commandes, `$ng0`, `$ng1` et `$ng2`, pour afficher les trois éléments sélectionnés le plus récemment. Après chaque instruction, la console affiche une référence de composant différente.">

### Sélectionner une directive ou un composant {#select-a-directive-or-component}

Comme dans les DevTools des navigateurs, vous pouvez inspecter la page pour sélectionner un composant ou une directive.
Cliquez sur l'icône **_Inspect element_** dans le coin supérieur gauche d'Angular DevTools et survolez un élément DOM de la page.
L'extension reconnaît les directives et/ou composants associés et vous permet de sélectionner l'élément correspondant dans l'arbre de composants.

<img src="assets/images/guide/devtools/inspect-element.png" alt="Une capture d'écran de l'onglet 'Components' avec une application de todo Angular visible. Tout en haut à gauche d'Angular DevTools, une icône représentant un écran contenant une souris est sélectionnée. La souris survole un élément todo dans l'interface de l'application Angular. L'élément est mis en évidence et une infobulle adjacente affiche le libellé `<TodoComponent>`.">

### Inspecter les vues différables {#inspect-deferrable-views}

En plus des directives, l'arbre des directives inclut aussi les [blocs `@defer`](/guide/templates/defer).

<img src="assets/images/guide/devtools/defer-block.png" />

Cliquer sur un bloc defer affiche plus de détails dans la barre latérale des propriétés : les différents blocs optionnels (par exemple `@loading`, `@placeholder` et `@error`), les déclencheurs configurés (déclencheurs defer, déclencheurs de préchargement et déclencheurs d'hydratation) et les options de timing comme les valeurs `minimum` et `after`.

### Hydratation {#hydration}

Lorsque l'[hydratation](/guide/hydration) est activée sur votre application SSR/SSG, l'arbre des directives affiche le statut d'hydratation de chaque composant.

En cas d'erreur, un message d'erreur s'affiche sur le composant concerné.

<img src="assets/images/guide/devtools/hydration-status.png" />

Le statut d'hydratation peut aussi être visualisé directement sur l'application en activant l'overlay.

<img src="assets/images/guide/devtools/hydration-overlay-ecom.png" />

Voici une illustration des overlays d'hydratation sur une application d'exemple de boutique en ligne Angular.

### Déboguer le graphe de signals {#debug-the-signal-graph}

Pour visualiser comment chaque signal, computed et effect d'un composant dépend des autres, consultez [Déboguer le graphe de signals](tools/devtools/signals).
