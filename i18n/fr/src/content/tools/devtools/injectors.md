<!-- source: 89b699c1ba391e4343f12792e566d03db5908aaf -->
# Inspecter vos injectors

NOTE: L'Injector Tree est disponible pour les applications Angular compilées avec la version 17 ou ultérieure.

## Afficher la hiérarchie des injectors de votre application {#view-the-injector-hierarchy-of-your-application}

L'onglet **Injector Tree** vous permet d'explorer la structure des injectors configurés dans votre application. Vous y verrez deux arbres représentant la [hiérarchie des injectors](guide/di/hierarchical-dependency-injection) de votre application. L'un est votre hiérarchie d'environnement, l'autre votre hiérarchie d'éléments.

<img src="assets/images/guide/devtools/di-injector-tree.png" alt="Une capture d'écran de l'onglet 'Profiler' montrant l'onglet Injector Tree d'Angular DevTools, qui visualise le graphe des injectors d'une application d'exemple.">

## Visualiser les chemins de résolution {#visualize-resolution-paths}

Lorsqu'un injector est sélectionné, le chemin que parcourt l'algorithme d'injection de dépendances d'Angular depuis cet injector jusqu'à la racine est mis en évidence. Pour les element injectors, cela inclut la mise en évidence des injectors d'environnement vers lesquels l'algorithme d'injection de dépendances saute lorsqu'une dépendance ne peut pas être résolue dans la hiérarchie d'éléments.

Consultez les [règles de résolution](guide/di/hierarchical-dependency-injection#resolution-rules) pour plus de détails sur la façon dont Angular détermine les chemins de résolution.

<img src="assets/images/guide/devtools/di-injector-tree-selected.png" alt="Une capture d'écran de l'onglet 'Profiler' montrant comment la visualisation de l'Injector Tree met en évidence les chemins de résolution lorsqu'un injector est sélectionné.">

## Afficher les providers d'un injector {#view-injector-providers}

Cliquer sur un injector dont des providers sont configurés affiche ces providers dans une liste à droite de la vue de l'arbre des injectors. Vous pouvez y voir le jeton fourni et son type. Le bouton à droite de chaque provider vous permet d'afficher le provider dans la console.

<img src="assets/images/guide/devtools/di-injector-tree-providers.png" alt="Une capture d'écran de l'onglet 'Profiler' montrant comment les providers deviennent visibles lorsqu'un injector est sélectionné.">
