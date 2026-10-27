<!-- source: 0efe9729d104fa4eb6e975ea93aae17e19c3282d -->
# Séquences d'animation complexes

IMPORTANT: Le package `@angular/animations` est désormais déprécié. L'équipe Angular recommande d'utiliser du CSS natif avec `animate.enter` et `animate.leave` pour les animations dans tout nouveau code. Pour en savoir plus, consultez le nouveau [guide des animations](/guide/animations) d'entrée et de sortie. Consultez aussi [Migrer depuis le package Animations d'Angular](guide/animations/migration) pour découvrir comment commencer à migrer vers des animations en CSS pur dans vos applications.

Jusqu'ici, nous avons vu des animations simples d'éléments HTML uniques.
Angular vous permet aussi d'animer des séquences coordonnées, comme une grille entière ou une liste d'éléments lorsqu'ils entrent dans une page ou en sortent.
Vous pouvez choisir d'exécuter plusieurs animations en parallèle, ou d'exécuter des animations distinctes de manière séquentielle, l'une après l'autre.

Les fonctions qui contrôlent les séquences d'animation complexes sont :

| Fonctions                         | Détails                                                        |
| :-------------------------------- | :------------------------------------------------------------- |
| `query()`                         | Trouve un ou plusieurs éléments HTML internes.                 |
| `stagger()`                       | Applique un délai en cascade aux animations de plusieurs éléments. |
| [`group()`](api/animations/group) | Exécute plusieurs étapes d'animation en parallèle.             |
| `sequence()`                      | Exécute des étapes d'animation l'une après l'autre.            |

## La fonction query() {#the-query-function}

La plupart des animations complexes s'appuient sur la fonction `query()` pour trouver des éléments enfants et leur appliquer des animations. Voici quelques exemples de base :

| Exemples                               | Détails                                                                                                                                                                                            |
| :------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `query()` suivi de `animate()`      | Permet de cibler des éléments HTML simples et de leur appliquer directement des animations.                                                                                                                          |
| `query()` suivi de `animateChild()` | Permet de cibler des éléments enfants, qui ont eux-mêmes des métadonnées d'animation, et de déclencher ces animations \(qui seraient sinon bloquées par l'animation de l'élément courant/parent\). |

Le premier argument de `query()` est une chaîne de [selector CSS](https://developer.mozilla.org/docs/Web/CSS/CSS_Selectors) qui peut aussi contenir les jetons spécifiques à Angular suivants :

| Jetons                       | Détails                                      |
| :------------------------- | :------------------------------------------- |
| `:enter` <br /> `:leave`   | Pour les éléments qui entrent/sortent.       |
| `:animating`               | Pour les éléments en cours d'animation.      |
| `@*` <br /> `@triggerName` | Pour les éléments ayant un déclencheur quelconque ou un déclencheur précis. |
| `:self`                    | L'élément animé lui-même.                    |

<docs-callout title="Éléments qui entrent et qui sortent">

Tous les éléments enfants ne sont pas réellement considérés comme entrants/sortants ; cela peut parfois être contre-intuitif et déroutant. Consultez la [documentation de l'API query](api/animations/query#entering-and-leaving-elements) pour plus d'informations.

Vous pouvez aussi en voir une illustration dans l'exemple d'animations \(présenté dans la [section d'introduction](guide/legacy-animations#about-this-guide) des animations\), sous l'onglet Querying.

</docs-callout>

## Animer plusieurs éléments avec les fonctions query() et stagger() {#animate-multiple-elements-using-query-and-stagger-functions}

Après avoir ciblé des éléments enfants avec `query()`, la fonction `stagger()` vous permet de définir un intervalle de temps entre chaque élément, pour les animer avec un délai entre eux.

L'exemple suivant montre comment utiliser les fonctions `query()` et `stagger()` pour animer une liste \(de héros\) en ajoutant chaque élément l'un après l'autre, avec un léger délai, de haut en bas.

- Utilisez `query()` pour rechercher un élément entrant dans la page qui répond à certains critères
- Pour chacun de ces éléments, utilisez `style()` pour lui appliquer le même style initial.
  Rendez-le transparent et utilisez `transform` pour le déplacer hors de sa position afin qu'il puisse glisser jusqu'à sa place.

- Utilisez `stagger()` pour retarder chaque animation de 30 millisecondes
- Animez chaque élément à l'écran pendant 0,5 seconde avec une courbe d'easing personnalisée, en le faisant apparaître en fondu et en annulant sa transformation simultanément

<docs-code header="hero-list-page.ts" path="adev/src/content/examples/animations/src/app/hero-list-page.ts" region="page-animations"/>

## Animation parallèle avec la fonction group() {#parallel-animation-using-group-function}

Vous avez vu comment ajouter un délai entre chaque animation successive.
Mais vous pouvez aussi vouloir configurer des animations qui se déroulent en parallèle.
Par exemple, vous pouvez vouloir animer deux propriétés CSS du même élément en utilisant une fonction d'`easing` différente pour chacune.
Pour cela, vous pouvez utiliser la fonction d'animation [`group()`](api/animations/group).

HELPFUL: La fonction [`group()`](api/animations/group) sert à regrouper des _étapes_ d'animation, et non des éléments animés.

L'exemple suivant utilise des [`group()`](api/animations/group) sur `:enter` et sur `:leave` pour deux configurations de timing différentes, appliquant ainsi deux animations indépendantes au même élément en parallèle.

<docs-code header="hero-list-groups.ts (extrait)" path="adev/src/content/examples/animations/src/app/hero-list-groups.ts" region="animationdef"/>

## Animations séquentielles ou parallèles {#sequential-vs-parallel-animations}

Les animations complexes peuvent impliquer beaucoup de choses en même temps.
Mais que faire si vous voulez créer une animation composée de plusieurs animations qui se succèdent ? Plus haut, vous avez utilisé [`group()`](api/animations/group) pour exécuter plusieurs animations en même temps, en parallèle.

Une deuxième fonction, `sequence()`, vous permet d'exécuter ces mêmes animations l'une après l'autre.
Dans `sequence()`, les étapes d'animation sont des appels aux fonctions `style()` ou `animate()`.

- Utilisez `style()` pour appliquer immédiatement les données de style fournies.
- Utilisez `animate()` pour appliquer les données de style sur un intervalle de temps donné.

## Exemple d'animation de filtre {#filter-animation-example}

Regardez une autre animation de la page d'exemple.
Sous l'onglet Filter/Stagger, saisissez du texte dans la zone de texte **Search Heroes**, par exemple `Magnet` ou `tornado`.

Le filtre fonctionne en temps réel pendant la saisie.
Des éléments quittent la page à chaque nouvelle lettre saisie, à mesure que le filtre devient plus strict.
La liste des héros réapparaît progressivement dans la page à mesure que vous effacez chaque lettre de la zone de filtre.

Le template HTML contient un déclencheur nommé `filterAnimation`.

<docs-code header="hero-list-page.html" path="adev/src/content/examples/animations/src/app/hero-list-page.html" region="filter-animations" language="angular-html"/>

Le `filterAnimation` du décorateur du composant contient trois transitions.

<docs-code header="hero-list-page.ts" path="adev/src/content/examples/animations/src/app/hero-list-page.ts" region="filter-animations"/>

Le code de cet exemple effectue les tâches suivantes :

- Ignore les animations quand l'utilisateur ouvre cette page pour la première fois ou y navigue \(l'animation de filtre réduit ce qui est déjà présent, elle ne fonctionne donc que sur des éléments qui existent déjà dans le DOM\)
- Filtre les héros selon la valeur du champ de recherche

Pour chaque changement :

- Masque un élément qui quitte le DOM en mettant son opacité et sa largeur à 0
- Anime un élément qui entre dans le DOM pendant 300 millisecondes.
  Pendant l'animation, l'élément reprend sa largeur et son opacité par défaut.

- S'il y a plusieurs éléments qui entrent dans le DOM ou en sortent, échelonne chaque animation en commençant par le haut de la page, avec un délai de 50 millisecondes entre chaque élément

## Animer les éléments d'une liste réordonnée {#animating-the-items-of-a-reordering-list}

Bien qu'Angular anime correctement les éléments d'une liste `*ngFor` par défaut, il n'en est plus capable si leur ordre change.
En effet, il perd la trace de quel élément est lequel, ce qui donne des animations cassées.
La seule façon d'aider Angular à suivre ces éléments est d'assigner une `TrackByFunction` à la directive `NgForOf`.
Ainsi, Angular sait toujours quel élément est lequel, ce qui lui permet d'appliquer à tout moment les bonnes animations aux bons éléments.

IMPORTANT: Si vous devez animer les éléments d'une liste `*ngFor` et que l'ordre de ces éléments risque de changer à l'exécution, utilisez toujours une `TrackByFunction`.

## Animations et encapsulation des vues des composants {#animations-and-component-view-encapsulation}

Les animations Angular reposent sur la structure DOM des composants et ne tiennent pas directement compte de l'[encapsulation des vues](guide/components/styling#style-scoping) : les composants utilisant `ViewEncapsulation.Emulated` se comportent donc exactement comme s'ils utilisaient `ViewEncapsulation.None` (`ViewEncapsulation.ShadowDom` et `ViewEncapsulation.ExperimentalIsolatedShadowDom` se comportent différemment, comme nous le verrons plus loin).

Par exemple, si la fonction `query()` (que vous verrez davantage dans la suite du guide des animations) était appliquée au sommet d'un arbre de composants utilisant l'encapsulation des vues émulée, cette query pourrait identifier (et donc animer) des éléments DOM à n'importe quelle profondeur de l'arbre.

En revanche, `ViewEncapsulation.ShadowDom` et `ViewEncapsulation.ExperimentalIsolatedShadowDom` modifient la structure DOM du composant en « masquant » des éléments DOM dans des éléments [`ShadowRoot`](https://developer.mozilla.org/docs/Web/API/ShadowRoot). Ces manipulations du DOM empêchent une partie de l'implémentation des animations de fonctionner correctement, car elle repose sur des structures DOM simples et ne tient pas compte des éléments `ShadowRoot`. Il est donc conseillé d'éviter d'appliquer des animations à des vues qui incluent des composants utilisant l'encapsulation des vues ShadowDom.

## Récapitulatif des séquences d'animation {#animation-sequence-summary}

Les fonctions Angular servant à animer plusieurs éléments commencent par `query()` pour trouver des éléments internes ; par exemple, rassembler toutes les images d'une `<div>`.
Les autres fonctions, `stagger()`, [`group()`](api/animations/group) et `sequence()`, appliquent des cascades ou vous permettent de contrôler la façon dont plusieurs étapes d'animation sont appliquées.

## En savoir plus sur les animations Angular {#more-on-angular-animations}

Ceci pourrait aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/legacy-animations" title="Introduction aux animations Angular"/>
  <docs-pill href="guide/legacy-animations/transition-and-triggers" title="Transition et déclencheurs"/>
  <docs-pill href="guide/legacy-animations/reusable-animations" title="Animations réutilisables"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
  <docs-pill href="guide/animations/migration" title="Migrer vers les animations CSS natives"/>
</docs-pill-row>
