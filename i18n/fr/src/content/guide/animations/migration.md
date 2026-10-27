<!-- source: 37ab500b89ade66e8313ff013ea3ea379f9f8184 -->
# Migrer depuis le package Animations d'Angular

Le package `@angular/animations` est déprécié depuis la v20.2, qui a aussi introduit la nouvelle fonctionnalité `animate.enter` et `animate.leave` pour ajouter des animations à votre application. Grâce à ces nouvelles fonctionnalités, vous pouvez remplacer toutes les animations basées sur `@angular/animations` par du CSS pur ou par des libraries d'animation JS. Retirer `@angular/animations` de votre application peut réduire considérablement la taille de votre bundle JavaScript. Les animations CSS natives offrent généralement de meilleures performances, car elles peuvent bénéficier de l'accélération matérielle. Ce guide présente la démarche pour refactoriser votre code de `@angular/animations` vers des animations CSS natives.

## Comment écrire des animations en CSS natif {#how-to-write-animations-in-native-css}

Si vous n'avez jamais écrit d'animations CSS natives, il existe de nombreux excellents guides pour bien démarrer. En voici quelques-uns :  
[Guide des animations CSS de MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animations/Using_CSS_animations)  
[Guide des animations CSS3 de W3Schools](https://www.w3schools.com/css/css3_animations.asp)  
[Le tutoriel complet sur les animations CSS](https://www.lambdatest.com/blog/css-animations-tutorial/)  
[Les animations CSS pour les débutants](https://thoughtbot.com/blog/css-animation-for-beginners)

ainsi que quelques vidéos :  
[Apprendre les animations CSS en 9 minutes](https://www.youtube.com/watch?v=z2LQYsZhsFw)  
[Playlist de tutoriels sur les animations CSS de Net Ninja](https://www.youtube.com/watch?v=jgw82b5Y2MU&list=PL4cUxeGkcC9iGYgmEd2dm3zAKzyCGDtM5)

Consultez certains de ces guides et tutoriels, puis revenez à ce guide.

## Créer des animations réutilisables {#creating-reusable-animations}

Comme avec le package animations, vous pouvez créer des animations réutilisables, partagées dans toute votre application. Avec le package animations, vous utilisiez pour cela la fonction `animation()` dans un fichier TypeScript partagé. La version en CSS natif est similaire, mais se trouve dans un fichier CSS partagé.

#### Avec le package Animations {#with-animations-package}

<docs-code header="animations.ts" path="adev/src/content/examples/animations/src/app/animations.1.ts" region="animation-example"/>

#### Avec du CSS natif {#with-native-css}

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-shared"/>

Ajouter la classe `animated-class` à un élément déclenche l'animation sur cet élément.

## Animer une transition {#animating-a-transition}

### Animer des états et des styles {#animating-state-and-styles}

Le package animations vous permettait de définir différents états avec la fonction [`state()`](api/animations/state) dans un composant. Par exemple, un état `open` ou `closed` contenant dans sa définition les styles propres à chacun de ces états. Par exemple :

#### Avec le package Animations {#animating-state-and-styles-with-animations-package}

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="state1"/>

Vous pouvez obtenir ce même comportement de manière native avec des classes CSS, soit avec une animation keyframes, soit avec un style de transition.

#### Avec du CSS natif {#animating-state-and-styles-with-native-css}

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-states"/>

Pour déclencher l'état `open` ou `closed`, il suffit de basculer des classes sur l'élément dans votre composant. Vous trouverez des exemples dans notre [guide des templates](guide/templates/binding#css-class-and-style-property-bindings).

Vous trouverez des exemples similaires dans le guide des templates pour [animer directement des styles](guide/templates/binding#css-style-properties).

### Transitions, timing et easing {#transitions-timing-and-easing}

La fonction `animate()` du package animations permet d'indiquer un timing, comme la durée, les délais et l'easing. Vous pouvez faire de même nativement en CSS avec plusieurs propriétés CSS ou propriétés raccourcies.

Spécifiez `animation-duration`, `animation-delay` et `animation-timing-function` pour une animation keyframes en CSS, ou utilisez plutôt la propriété raccourcie `animation`.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-timing"/>

De même, vous pouvez utiliser `transition-duration`, `transition-delay` et `transition-timing-function`, ainsi que le raccourci `transition`, pour les animations qui n'utilisent pas `@keyframes`.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="transition-timing"/>

### Déclencher une animation {#triggering-an-animation}

Le package animations exigeait de spécifier des déclencheurs avec la fonction `trigger()` et d'y imbriquer tous vos états. Avec du CSS natif, c'est inutile. Les animations peuvent être déclenchées en basculant des styles ou des classes CSS. Dès qu'une classe est présente sur un élément, l'animation se produit. Retirer la classe ramène l'élément au CSS défini pour lui. Il faut ainsi beaucoup moins de code pour réaliser la même animation. Voici un exemple :

#### Avec le package Animations {#triggering-an-animation-with-animations-package}

<docs-code-multifile>
    <docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/animations-package/open-close.ts" />
    <docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/animations-package/open-close.html" />
    <docs-code header="open-close.css" path="adev/src/content/examples/animations/src/app/animations-package/open-close.css"/>
</docs-code-multifile>

#### Avec du CSS natif {#triggering-an-animation-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/open-close.ts">
    <docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/native-css/open-close.ts" />
    <docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/native-css/open-close.html" />
    <docs-code header="open-close.css" path="adev/src/content/examples/animations/src/app/native-css/open-close.css"/>
</docs-code-multifile>

## Transition et déclencheurs {#transition-and-triggers}

### États prédéfinis et correspondance par wildcard {#predefined-state-and-wildcard-matching}

Le package animations permet de faire correspondre vos états définis à une transition au moyen de chaînes. Par exemple, une animation de l'état ouvert vers l'état fermé s'écrit `open => closed`. Vous pouvez utiliser des wildcards pour faire correspondre n'importe quel état à un état cible, comme `* => closed`, et le mot-clé `void` peut être utilisé pour les états d'entrée et de sortie. Par exemple : `* => void` quand un élément quitte une vue, ou `void => *` quand l'élément entre dans une vue.

Ces patterns de correspondance d'états sont totalement inutiles quand vous animez directement avec CSS. Vous pouvez gérer quelles transitions et quelles animations `@keyframes` s'appliquent selon les classes et / ou les styles que vous définissez sur les éléments. Vous pouvez aussi ajouter `@starting-style` pour contrôler l'apparence de l'élément dès son entrée dans le DOM.

### Calcul automatique des propriétés avec des wildcards {#automatic-property-calculation-with-wildcards}

Le package animations permet d'animer des choses historiquement difficiles à animer, comme le passage d'une hauteur fixe à `height: auto`. Vous pouvez désormais le faire aussi en CSS pur.

#### Avec le package Animations {#automatic-property-calculation-with-animations-package}

<docs-code-multifile>
    <docs-code header="auto-height.ts" path="adev/src/content/examples/animations/src/app/animations-package/auto-height.ts" />
    <docs-code header="auto-height.html" path="adev/src/content/examples/animations/src/app/animations-package/auto-height.html" />
    <docs-code header="auto-height.css" path="adev/src/content/examples/animations/src/app/animations-package/auto-height.css" />
</docs-code-multifile>

Vous pouvez utiliser CSS Grid pour animer vers une hauteur auto.

#### Avec du CSS natif {#automatic-property-calculation-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/auto-height.ts">
    <docs-code header="auto-height.ts" path="adev/src/content/examples/animations/src/app/native-css/auto-height.ts" />
    <docs-code header="auto-height.html" path="adev/src/content/examples/animations/src/app/native-css/auto-height.html" />
    <docs-code header="auto-height.css" path="adev/src/content/examples/animations/src/app/native-css/auto-height.css"  />
</docs-code-multifile>

Si vous n'avez pas à vous soucier de la prise en charge de tous les navigateurs, vous pouvez aussi regarder `calc-size()`, qui est la véritable solution pour animer une hauteur auto. Consultez [la documentation de MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/calc-size) et (ce tutoriel)[https://frontendmasters.com/blog/one-of-the-boss-battles-of-css-is-almost-won-transitioning-to-auto/] pour plus d'informations.

### Animer l'entrée et la sortie d'une vue {#animate-entering-and-leaving-a-view}

Le package animations proposait la correspondance de patterns évoquée plus haut pour l'entrée et la sortie, mais incluait aussi les alias raccourcis `:enter` et `:leave`.

#### Avec le package Animations {#enter-and-leave-with-animations-package}

<docs-code-multifile>
    <docs-code header="insert-remove.ts" path="adev/src/content/examples/animations/src/app/animations-package/insert-remove.ts" />
    <docs-code header="insert-remove.html" path="adev/src/content/examples/animations/src/app/animations-package/insert-remove.html" />
    <docs-code header="insert-remove.css" path="adev/src/content/examples/animations/src/app/animations-package/insert-remove.css" />
</docs-code-multifile>

#### Avec du CSS natif {#enter-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/insert.ts">
    <docs-code header="insert.ts" path="adev/src/content/examples/animations/src/app/native-css/insert.ts" />
    <docs-code header="insert.html" path="adev/src/content/examples/animations/src/app/native-css/insert.html" />
    <docs-code header="insert.css" path="adev/src/content/examples/animations/src/app/native-css/insert.css"  />
</docs-code-multifile>

#### Avec du CSS natif {#leave-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/remove.ts">
    <docs-code header="remove.ts" path="adev/src/content/examples/animations/src/app/native-css/remove.ts" />
    <docs-code header="remove.html" path="adev/src/content/examples/animations/src/app/native-css/remove.html" />
    <docs-code header="remove.css" path="adev/src/content/examples/animations/src/app/native-css/remove.css"  />
</docs-code-multifile>

Pour plus d'informations sur `animate.enter` et `animate.leave`, consultez le [guide des animations d'entrée et de sortie](guide/animations).

### Animer une incrémentation et une décrémentation {#animating-increment-and-decrement}

En plus des `:enter` et `:leave` évoqués plus haut, il existe aussi `:increment` et `:decrement`. Vous pouvez également les animer en ajoutant et en retirant des classes. Contrairement aux alias intégrés du package animations, les classes ne sont pas appliquées automatiquement quand les valeurs augmentent ou diminuent. Vous pouvez appliquer les classes appropriées par programmation. Voici un exemple :

#### Avec le package Animations {#increment-and-decrement-with-animations-package}

<docs-code-multifile>
    <docs-code header="increment-decrement.ts" path="adev/src/content/examples/animations/src/app/animations-package/increment-decrement.ts" />
    <docs-code header="increment-decrement.html" path="adev/src/content/examples/animations/src/app/animations-package/increment-decrement.html" />
    <docs-code header="increment-decrement.css" path="adev/src/content/examples/animations/src/app/animations-package/increment-decrement.css" />
</docs-code-multifile>

#### Avec du CSS natif {#increment-and-decrement-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.ts">
    <docs-code header="increment-decrement.ts" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.ts" />
    <docs-code header="increment-decrement.html" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.html" />
    <docs-code header="increment-decrement.css" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.css" />
</docs-code-multifile>

### Animations parent / enfant {#parent---child-animations}

Contrairement au package animations, quand plusieurs animations sont définies dans un même composant, aucune animation n'a la priorité sur une autre et rien n'empêche une animation de se déclencher. Tout enchaînement d'animations doit être géré dans la définition de votre animation CSS, avec des délais d'animation / de transition, et / ou avec `animationend` ou `transitionend` pour ajouter le CSS suivant à animer.

Les animations enfants ne se déclenchent qu'au sein du template d'un même composant. Dans le package `@angular/animations`, les animations parentes pouvaient cibler et déclencher des animations dans des composants enfants imbriqués avec `query()` et `animateChild()`. Avec `animate.leave` et les animations CSS natives, les animations définies dans les templates des composants enfants imbriqués ne se déclenchent pas quand un composant parent retire un élément ou une vue. Seules les animations imbriquées au sein du template d'un même composant Angular s'exécutent. Consultez le [guide des animations d'entrée et de sortie](guide/animations#element-removal-order) pour plus d'informations à ce sujet.

### Désactiver une animation ou toutes les animations {#disabling-an-animation-or-all-animations}

Avec les animations CSS natives, si vous souhaitez désactiver les animations que vous avez définies, plusieurs options s'offrent à vous.

1. Créez une classe personnalisée qui force animation et transition à `none`.

```css
.no-animation {
  animation: none !important;
  transition: none !important;
}
```

Appliquer cette classe à un élément empêche toute animation de se déclencher sur cet élément. Vous pouvez aussi l'appliquer à tout votre DOM ou à une section de votre DOM pour imposer ce comportement. Cependant, cela empêche les événements d'animation de se déclencher. Si vous attendez des événements d'animation pour retirer un élément, cette solution ne fonctionnera pas. Une solution de contournement consiste à définir plutôt des durées de 1 milliseconde.

2. Utilisez la media query [`prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) pour qu'aucune animation ne soit jouée pour les utilisateurs qui préfèrent moins d'animations.

3. Évitez d'ajouter des classes d'animation par programmation

### Callbacks d'animation {#animation-callbacks}

Le package animations exposait des callbacks à utiliser si vous vouliez faire quelque chose à la fin de l'animation. Les animations CSS natives disposent aussi de ces callbacks.

[`OnAnimationStart`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animationstart_event)  
[`OnAnimationEnd`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animationend_event)  
[`OnAnimationIteration`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animationiteration_event)  
[`OnAnimationCancel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animationcancel_event)

[`OnTransitionStart`](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionstart_event)  
[`OnTransitionRun`](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionrun_event)  
[`OnTransitionEnd`](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event)  
[`OnTransitionCancel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitioncancel_event)

La Web Animations API offre de nombreuses fonctionnalités supplémentaires. [Consultez la documentation](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API) pour voir toutes les API d'animation disponibles.

NOTE: Faites attention aux problèmes de propagation (bubbling) avec ces callbacks. Si vous animez des enfants et des parents, les événements remontent des enfants vers les parents. Envisagez d'arrêter la propagation ou d'examiner plus en détail l'événement pour déterminer si vous répondez bien à la cible d'événement voulue plutôt qu'à un événement remonté depuis un nœud enfant. Vous pouvez examiner la propriété `animationname` ou les propriétés en cours de transition pour vérifier que vous ciblez les bons nœuds.

## Séquences complexes {#complex-sequences}

Le package animations intègre des fonctionnalités pour créer des séquences complexes. Toutes ces séquences sont parfaitement réalisables sans le package animations.

### Cibler des éléments précis {#targeting-specific-elements}

Dans le package animations, vous pouviez cibler des éléments précis avec la fonction `query()` pour trouver des éléments par nom de classe CSS, de façon similaire à [`document.querySelector()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelector). C'est inutile avec les animations CSS natives. Vous pouvez plutôt utiliser vos selectors CSS pour cibler des sous-classes et appliquer la `transform` ou l'`animation` souhaitée.

Pour basculer des classes sur des nœuds enfants dans un template, vous pouvez utiliser des bindings de classe et de style afin d'ajouter les animations aux bons moments.

### Stagger() {#stagger}

La fonction `stagger()` vous permettait de retarder l'animation de chaque élément d'une liste d'une durée donnée pour créer un effet de cascade. Vous pouvez reproduire ce comportement en CSS natif avec `animation-delay` ou `transition-delay`. Voici un exemple de ce à quoi ce CSS pourrait ressembler.

#### Avec le package Animations {#stagger-with-animations-package}

<docs-code-multifile>
    <docs-code header="stagger.ts" path="adev/src/content/examples/animations/src/app/animations-package/stagger.ts" />
    <docs-code header="stagger.html" path="adev/src/content/examples/animations/src/app/animations-package/stagger.html" />
    <docs-code header="stagger.css" path="adev/src/content/examples/animations/src/app/animations-package/stagger.css" />
</docs-code-multifile>

#### Avec du CSS natif {#stagger-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/stagger.ts">
    <docs-code header="stagger.ts" path="adev/src/content/examples/animations/src/app/native-css/stagger.ts" />
    <docs-code header="stagger.html" path="adev/src/content/examples/animations/src/app/native-css/stagger.html" />
    <docs-code header="stagger.css" path="adev/src/content/examples/animations/src/app/native-css/stagger.css" />
</docs-code-multifile>

### Animations parallèles {#parallel-animations}

Le package animations propose une fonction `group()` pour jouer plusieurs animations en même temps. En CSS, vous avez un contrôle total sur le timing des animations. Si vous avez défini plusieurs animations, vous pouvez toutes les appliquer en même temps.

```css
.target-element {
  animation:
    rotate 3s,
    fade-in 2s;
}
```

Dans cet exemple, les animations `rotate` et `fade-in` se déclenchent en même temps.

### Animer les éléments d'une liste réordonnée {#animating-the-items-of-a-reordering-list}

Le réordonnancement des éléments d'une liste fonctionne d'emblée avec les techniques décrites précédemment. Aucun travail spécifique supplémentaire n'est nécessaire. Les éléments d'une boucle `@for` sont correctement retirés puis rajoutés, ce qui déclenche les animations utilisant `@starting-styles` pour les animations d'entrée. Vous pouvez aussi utiliser `animate.enter` pour obtenir le même comportement. Utilisez `animate.leave` pour animer les éléments lorsqu'ils sont retirés, comme dans l'exemple ci-dessus.

#### Avec le package Animations {#reordering-list-with-animations-package}

<docs-code-multifile>
    <docs-code header="reorder.ts" path="adev/src/content/examples/animations/src/app/animations-package/reorder.ts" />
    <docs-code header="reorder.html" path="adev/src/content/examples/animations/src/app/animations-package/reorder.html" />
    <docs-code header="reorder.css" path="adev/src/content/examples/animations/src/app/animations-package/reorder.css" />
</docs-code-multifile>

#### Avec du CSS natif {#reordering-list-with-native-css}

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/reorder.ts">
    <docs-code header="reorder.ts" path="adev/src/content/examples/animations/src/app/native-css/reorder.ts" />
    <docs-code header="reorder.html" path="adev/src/content/examples/animations/src/app/native-css/reorder.html" />
    <docs-code header="reorder.css" path="adev/src/content/examples/animations/src/app/native-css/reorder.css" />
</docs-code-multifile>

## Migrer les usages d'AnimationPlayer {#migrating-usages-of-animationplayer}

La classe `AnimationPlayer` donne accès à une animation pour réaliser des opérations plus avancées, comme mettre en pause, lancer, redémarrer et terminer une animation depuis le code. Tout cela peut aussi être géré nativement.

Vous pouvez récupérer directement les animations d'un élément avec [`Element.getAnimations()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAnimations). Cette méthode renvoie un tableau de toutes les [`Animation`](https://developer.mozilla.org/en-US/docs/Web/API/Animation) de cet élément. Vous pouvez utiliser l'API `Animation` pour faire bien plus que ce que permettait l'`AnimationPlayer` du package animations. Vous pouvez ensuite appeler `cancel()`, `play()`, `pause()`, `reverse()` et bien d'autres. Cette API native devrait vous fournir tout ce dont vous avez besoin pour contrôler vos animations.

## Transitions de route {#route-transitions}

Vous pouvez utiliser les view transitions pour animer le passage d'une route à l'autre. Consultez le [guide des animations de transition de route](guide/routing/route-transition-animations) pour commencer.
