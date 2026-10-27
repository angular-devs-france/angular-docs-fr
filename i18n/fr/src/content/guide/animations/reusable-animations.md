<!-- source: ab37f5840246b7a6a12af4147f4526f516a3e282 -->
# Animations réutilisables

IMPORTANT: Le package `@angular/animations` est désormais déprécié. L'équipe Angular recommande d'utiliser le CSS natif avec `animate.enter` et `animate.leave` pour les animations dans tout nouveau code. Pour en savoir plus, consultez le nouveau [guide des animations](guide/animations) d'entrée et de sortie. Consultez aussi [Abandonner le package Animations d'Angular](guide/animations/migration) pour savoir comment commencer à migrer vers des animations en CSS pur dans vos applications.

Cette page fournit des exemples de création d'animations réutilisables.

## Créer des animations réutilisables {#create-reusable-animations}

Pour créer une animation réutilisable, utilisez la fonction [`animation()`](api/animations/animation) afin de définir une animation dans un fichier `.ts` distinct, et déclarez cette définition d'animation comme une variable `const` exportée.
Vous pouvez ensuite importer et réutiliser cette animation dans les composants de votre application à l'aide de la fonction [`useAnimation()`](api/animations/useAnimation).

<docs-code header="animations.ts" path="adev/src/content/examples/animations/src/app/animations.1.ts" region="animation-const"/>

Dans l'extrait de code précédent, `transitionAnimation` devient réutilisable parce qu'il est déclaré comme variable exportée.

HELPFUL: Les entrées `height`, `opacity`, `backgroundColor` et `time` sont remplacées à l'exécution.

Vous pouvez aussi exporter une partie d'une animation.
Par exemple, l'extrait suivant exporte le `trigger` de l'animation.

<docs-code header="animations.ts" path="adev/src/content/examples/animations/src/app/animations.1.ts" region="trigger-const"/>

À partir de là, vous pouvez importer des variables d'animation réutilisables dans la classe de votre composant.
Par exemple, l'extrait de code suivant importe la variable `transitionAnimation` et l'utilise via la fonction `useAnimation()`.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.3.ts" region="reusable"/>

## En savoir plus sur les animations Angular {#more-on-angular-animations}

Les pages suivantes pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/legacy-animations" title="Introduction aux animations Angular"/>
  <docs-pill href="guide/legacy-animations/transition-and-triggers" title="Transitions et déclencheurs"/>
  <docs-pill href="guide/legacy-animations/complex-sequences" title="Séquences d'animation complexes"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
  <docs-pill href="guide/animations/migration" title="Migrer vers les animations CSS natives"/>
</docs-pill-row>
