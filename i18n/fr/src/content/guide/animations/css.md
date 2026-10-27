<!-- source: a32f7cbc564e2d7c3ede67d9b7ae6d82c3dc1c69 -->
# Animer votre application avec CSS

CSS offre un ensemble d'outils robustes pour créer des animations élégantes et attrayantes dans votre application.

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

Vous pouvez créer des animations réutilisables, partagées dans toute votre application, avec `@keyframes`. Définissez des animations keyframes dans un fichier CSS partagé, et vous pourrez réutiliser ces animations keyframes où vous le souhaitez dans votre application.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-shared"/>

Ajouter la classe `animated-class` à un élément déclenche l'animation sur cet élément.

## Animer une transition {#animating-a-transition}

### Animer des états et des styles {#animating-state-and-styles}

Vous pouvez vouloir animer le passage entre deux états différents, par exemple quand un élément est ouvert ou fermé. Vous pouvez y parvenir avec des classes CSS, soit avec une animation keyframes, soit avec un style de transition.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-states"/>

Pour déclencher l'état `open` ou `closed`, il suffit de basculer des classes sur l'élément dans votre composant. Vous trouverez des exemples dans notre [guide des templates](guide/templates/binding#css-class-and-style-property-bindings).

Vous trouverez des exemples similaires dans le guide des templates pour [animer directement des styles](guide/templates/binding#css-style-properties).

### Transitions, timing et easing {#transitions-timing-and-easing}

Animer nécessite souvent d'ajuster le timing, les délais et le comportement d'easing. Vous pouvez le faire avec plusieurs propriétés CSS ou propriétés raccourcies.

Spécifiez `animation-duration`, `animation-delay` et `animation-timing-function` pour une animation keyframes en CSS, ou utilisez plutôt la propriété raccourcie `animation`.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="animation-timing"/>

De même, vous pouvez utiliser `transition-duration`, `transition-delay` et `transition-timing-function`, ainsi que le raccourci `transition`, pour les animations qui n'utilisent pas `@keyframes`.

<docs-code header="animations.css" path="adev/src/content/examples/animations/src/app/animations.css" region="transition-timing"/>

### Déclencher une animation {#triggering-an-animation}

Les animations peuvent être déclenchées en basculant des styles ou des classes CSS. Dès qu'une classe est présente sur un élément, l'animation se produit. Retirer la classe ramène l'élément au CSS défini pour lui. Voici un exemple :

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/open-close.ts">
    <docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/native-css/open-close.ts" />
    <docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/native-css/open-close.html" />
    <docs-code header="open-close.css" path="adev/src/content/examples/animations/src/app/native-css/open-close.css"/>
</docs-code-multifile>

## Transition et déclencheurs {#transition-and-triggers}

### Animer une hauteur auto {#animating-auto-height}

Vous pouvez utiliser CSS Grid pour animer vers une hauteur auto.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/auto-height.ts">
    <docs-code header="auto-height.ts" path="adev/src/content/examples/animations/src/app/native-css/auto-height.ts" />
    <docs-code header="auto-height.html" path="adev/src/content/examples/animations/src/app/native-css/auto-height.html" />
    <docs-code header="auto-height.css" path="adev/src/content/examples/animations/src/app/native-css/auto-height.css"  />
</docs-code-multifile>

Si vous n'avez pas à vous soucier de la prise en charge de tous les navigateurs, vous pouvez aussi regarder `calc-size()`, qui est la véritable solution pour animer une hauteur auto. Consultez [la documentation de MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/calc-size) et [ce tutoriel](https://frontendmasters.com/blog/one-of-the-boss-battles-of-css-is-almost-won-transitioning-to-auto/) pour plus d'informations.

### Animer l'entrée et la sortie d'une vue {#animate-entering-and-leaving-a-view}

Vous pouvez créer des animations pour le moment où un élément entre dans une vue ou en sort. Commençons par voir comment animer un élément qui entre dans une vue. Nous allons utiliser `animate.enter`, qui applique des classes d'animation quand un élément entre dans la vue.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/insert.ts">
    <docs-code header="insert.ts" path="adev/src/content/examples/animations/src/app/native-css/insert.ts" />
    <docs-code header="insert.html" path="adev/src/content/examples/animations/src/app/native-css/insert.html" />
    <docs-code header="insert.css" path="adev/src/content/examples/animations/src/app/native-css/insert.css"  />
</docs-code-multifile>

Animer un élément quand il quitte la vue ressemble à l'animation de son entrée dans la vue. Utilisez `animate.leave` pour indiquer quelles classes CSS appliquer quand l'élément quitte la vue.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/remove.ts">
    <docs-code header="remove.ts" path="adev/src/content/examples/animations/src/app/native-css/remove.ts" />
    <docs-code header="remove.html" path="adev/src/content/examples/animations/src/app/native-css/remove.html" />
    <docs-code header="remove.css" path="adev/src/content/examples/animations/src/app/native-css/remove.css"  />
</docs-code-multifile>

NOTE: Les animations `animate.leave` des enfants ne se déclenchent qu'au sein du template d'un même composant. Les animations `animate.leave` des composants imbriqués ne se déclenchent pas quand un élément parent est retiré.

Pour plus d'informations sur `animate.enter` et `animate.leave`, consultez le [guide des animations d'entrée et de sortie](guide/animations).

### Animer une incrémentation et une décrémentation {#animating-increment-and-decrement}

Animer lors d'une incrémentation ou d'une décrémentation est un pattern courant dans les applications. Voici un exemple de la façon d'obtenir ce comportement.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.ts">
    <docs-code header="increment-decrement.ts" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.ts" />
    <docs-code header="increment-decrement.html" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.html" />
    <docs-code header="increment-decrement.css" path="adev/src/content/examples/animations/src/app/native-css/increment-decrement.css" />
</docs-code-multifile>

### Désactiver une animation ou toutes les animations {#disabling-an-animation-or-all-animations}

Si vous souhaitez désactiver les animations que vous avez définies, plusieurs options s'offrent à vous.

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

Si vous souhaitez exécuter des actions à certains moments d'une animation, vous pouvez écouter un certain nombre d'événements. En voici quelques-uns.

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

Les animations sont souvent plus compliquées qu'un simple fondu en entrée ou en sortie. Vous pouvez avoir de nombreuses séquences d'animations complexes à exécuter. Examinons quelques-uns de ces scénarios possibles.

### Échelonner les animations dans une liste {#staggering-animations-in-a-list}

Un effet courant consiste à échelonner les animations de chaque élément d'une liste pour créer un effet de cascade. Vous pouvez y parvenir avec `animation-delay` ou `transition-delay`. Voici un exemple de ce à quoi ce CSS pourrait ressembler.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/stagger.ts">
    <docs-code header="stagger.ts" path="adev/src/content/examples/animations/src/app/native-css/stagger.ts" />
    <docs-code header="stagger.html" path="adev/src/content/examples/animations/src/app/native-css/stagger.html" />
    <docs-code header="stagger.css" path="adev/src/content/examples/animations/src/app/native-css/stagger.css" />
</docs-code-multifile>

### Animations parallèles {#parallel-animations}

Vous pouvez appliquer plusieurs animations à un élément en même temps avec la propriété raccourcie `animation`. Chacune peut avoir ses propres durées et délais. Cela vous permet de composer des animations et de créer des effets complexes.

```css
.target-element {
  animation:
    rotate 3s,
    fade-in 2s;
}
```

Dans cet exemple, les animations `rotate` et `fade-in` se déclenchent en même temps, mais ont des durées différentes.

### Animer les éléments d'une liste réordonnée {#animating-the-items-of-a-reordering-list}

Les éléments d'une boucle `@for` sont retirés puis rajoutés, ce qui déclenche les animations utilisant `@starting-styles` pour les animations d'entrée. Vous pouvez aussi utiliser `animate.enter` pour obtenir le même comportement. Utilisez `animate.leave` pour animer les éléments lorsqu'ils sont retirés, comme dans l'exemple ci-dessous.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/native-css/reorder.ts">
    <docs-code header="reorder.ts" path="adev/src/content/examples/animations/src/app/native-css/reorder.ts" />
    <docs-code header="reorder.html" path="adev/src/content/examples/animations/src/app/native-css/reorder.html" />
    <docs-code header="reorder.css" path="adev/src/content/examples/animations/src/app/native-css/reorder.css" />
</docs-code-multifile>

## Contrôle des animations par programmation {#programmatic-control-of-animations}

Vous pouvez récupérer directement les animations d'un élément avec [`Element.getAnimations()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAnimations). Cette méthode renvoie un tableau de toutes les [`Animation`](https://developer.mozilla.org/en-US/docs/Web/API/Animation) de cet élément. Vous pouvez utiliser l'API `Animation` pour faire bien plus que ce que permettait l'`AnimationPlayer` du package animations. Vous pouvez ensuite appeler `cancel()`, `play()`, `pause()`, `reverse()` et bien d'autres. Cette API native devrait vous fournir tout ce dont vous avez besoin pour contrôler vos animations.

## En savoir plus sur les animations Angular {#more-on-angular-animations}

Ceci pourrait aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/animations" title="Animations d'entrée et de sortie"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
</docs-pill-row>
