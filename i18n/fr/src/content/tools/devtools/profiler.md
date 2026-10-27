<!-- source: b3ec6dbd7731ceeb4be016fc9784b13aabf05443 -->
# Profiler votre application

L'onglet **Profiler** vous permet de visualiser l'exécution de la détection de changements d'Angular.
C'est utile pour déterminer quand et comment la détection de changements affecte les performances de votre application.

<img src="assets/images/guide/devtools/profiler.png" alt="Une capture d'écran de l'onglet 'Profiler' qui indique 'Click the play button to start a new recording, or upload a json file containing profiler data.' À côté se trouvent un bouton d'enregistrement pour commencer l'enregistrement d'un nouveau profil, ainsi qu'un sélecteur de fichier pour choisir un profil existant.">

L'onglet Profiler vous permet de commencer à profiler l'application courante ou d'importer un profil existant issu d'une exécution précédente.
Pour commencer à profiler votre application, survolez le cercle dans le coin supérieur gauche de l'onglet **Profiler** et cliquez sur **Start recording**.

Pendant le profilage, Angular DevTools capture les événements d'exécution, comme la détection de changements et l'exécution de chaque lifecycle hook.
Interagissez avec votre application pour déclencher la détection de changements et générer des données exploitables par Angular DevTools.
Pour terminer l'enregistrement, cliquez de nouveau sur le cercle pour **Stop recording**.

Vous pouvez aussi importer un enregistrement existant.
Pour en savoir plus sur cette fonctionnalité, consultez la section [Importer un enregistrement](tools/devtools/profiler#import-and-export-recordings).

## Comprendre l'exécution de votre application {#understand-your-applications-execution}

Après l'enregistrement ou l'import d'un profil, Angular DevTools affiche une visualisation des cycles de détection de changements.

<img src="assets/images/guide/devtools/default-profiler-view.png" alt="Une capture d'écran de l'onglet 'Profiler' après l'enregistrement ou l'import d'un profil. Elle affiche un diagramme en barres illustrant différents cycles de détection de changements, avec un texte indiquant 'Select a bar to preview a particular change detection cycle'.">

Chaque barre de la séquence représente un cycle de détection de changements dans votre application.
Plus une barre est haute, plus l'application a passé de temps à exécuter la détection de changements pendant ce cycle.
Lorsque vous sélectionnez une barre, DevTools affiche des informations utiles à son sujet, notamment :

- Un diagramme en barres avec tous les composants et directives capturés pendant ce cycle
- Le temps passé par Angular à exécuter la détection de changements pendant ce cycle.
- Une estimation de la fréquence d'images perçue par l'utilisateur (si elle est inférieure à 60 fps)

<img src="assets/images/guide/devtools/profiler-selected-bar.png" alt="Une capture d'écran de l'onglet 'Profiler'. L'utilisateur a sélectionné une barre et un menu déroulant à proximité affiche 'Bar chart', avec un second diagramme en barres en dessous. Le nouveau diagramme comporte deux barres qui occupent la majeure partie de l'espace, l'une intitulée `TodosComponent` et l'autre `NgForOf`. Les autres barres sont si petites qu'elles sont négligeables en comparaison.">

## Comprendre l'exécution des composants {#understand-component-execution}

Le diagramme en barres affiché après avoir cliqué sur un cycle de détection de changements présente une vue détaillée du temps passé par votre application à exécuter la détection de changements dans ce composant ou cette directive.

Cet exemple montre le temps total passé par la directive `NgForOf` et la méthode qui a été appelée sur celle-ci.

<img src="assets/images/guide/devtools/directive-details.png" alt="Une capture d'écran de l'onglet 'Profiler' où la barre `NgForOf` est sélectionnée. Une vue détaillée de `NgForOf` est visible à droite et indique 'Total time spent: 1.76 ms'. Elle comprend un tableau d'exactement une ligne, qui liste `NgForOf` comme directive avec une méthode `ngDoCheck` ayant pris 1,76 ms. Elle comprend aussi une liste intitulée 'Parent Hierarchy' contenant les composants parents de cette directive.">

## Vues hiérarchiques {#hierarchical-views}

<img src="assets/images/guide/devtools/flame-graph-view.png" alt="Une capture d'écran de l'onglet 'Profiler'. L'utilisateur a sélectionné une barre et un menu déroulant à proximité affiche désormais 'Flame graph', avec un flame graph en dessous. Le flame graph commence par une ligne appelée 'Entire application' et une autre appelée 'AppComponent'. En dessous, les lignes se divisent en plusieurs éléments, à commencer par `[RouterOutlet]` et `DemoAppComponent` sur la troisième ligne. Quelques niveaux plus bas, une cellule est surlignée en rouge.">

Vous pouvez aussi visualiser l'exécution de la détection de changements dans une vue de type flame graph.

Chaque tuile du graphe représente un élément à l'écran à une position donnée dans l'arbre de rendu.
Prenons par exemple un cycle de détection de changements où un `LoggedOutUserComponent` est supprimé et où Angular affiche un `LoggedInUserComponent` à sa place. Dans ce scénario, les deux composants seront affichés dans la même tuile.

L'axe des x représente le temps total nécessaire au rendu de ce cycle de détection de changements.
L'axe des y représente la hiérarchie des éléments. Exécuter la détection de changements pour un élément nécessite d'effectuer le rendu de ses directives et de ses composants enfants.
Ensemble, ce graphe permet de visualiser quels composants prennent le plus de temps à s'afficher et où ce temps est passé.

Chaque tuile est colorée en fonction du temps qu'Angular y a passé.
Angular DevTools détermine l'intensité de la couleur à partir du temps passé, relativement à la tuile dont le rendu a pris le plus de temps.

Lorsque vous cliquez sur une tuile, vous voyez ses détails dans le panneau de droite.
Double-cliquer sur la tuile zoome dessus, ce qui vous permet de voir plus facilement ses enfants imbriqués.

## Déboguer la détection de changements et les composants `OnPush` {#debug-change-detection-and-onpush-components}

Normalement, le graphe visualise le temps nécessaire au _rendu_ d'une application, pour une frame de détection de changements donnée. Cependant, certains composants, comme les composants `OnPush`, ne sont rendus de nouveau que si leurs propriétés d'input changent. Il peut être utile de visualiser le flame graph sans ces composants pour certaines frames.

Pour visualiser uniquement les composants d'une frame de détection de changements qui sont passés par le processus de détection de changements, cochez la case **Change detection** en haut, au-dessus du flame graph.

Cette vue met en évidence tous les composants passés par la détection de changements et affiche en gris ceux qui n'y sont pas passés, comme les composants `OnPush` qui n'ont pas été rendus de nouveau.

<img src="assets/images/guide/devtools/debugging-onpush.png" alt="Une capture d'écran de l'onglet 'Profiler' affichant une visualisation en flame chart d'un cycle de détection de changements. Une case intitulée 'Show only change detection' est maintenant cochée. Le flame graph ressemble beaucoup au précédent, mais la couleur des composants est passée de l'orange au bleu. Plusieurs tuiles intitulées `[RouterOutlet]` ne sont plus colorées.">

## Importer et exporter des enregistrements {#import-and-export-recordings}

Cliquez sur le bouton **Save Profile** en haut à droite d'une session de profilage enregistrée pour l'exporter sous forme de fichier JSON et l'enregistrer sur le disque.
Plus tard, importez le fichier dans la vue initiale du profiler en cliquant sur le champ **Choose file**.

<img src="assets/images/guide/devtools/save-profile.png" alt="Une capture d'écran de l'onglet 'Profiler' affichant des cycles de détection de changements. Un bouton 'Save Profile' est visible sur la droite.">
