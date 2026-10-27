<!-- source: 7cfe75968dee8990b3342fb034a3d9f1c4b18c4f -->
# Transitions et déclencheurs d'animation

IMPORTANT: Le package `@angular/animations` est désormais déprécié. L'équipe Angular recommande d'utiliser le CSS natif avec `animate.enter` et `animate.leave` pour les animations dans tout nouveau code. Pour en savoir plus, consultez le nouveau [guide des animations](guide/animations) d'entrée et de sortie. Consultez aussi [Abandonner le package Animations d'Angular](guide/animations/migration) pour savoir comment commencer à migrer vers des animations en CSS pur dans vos applications.

Ce guide approfondit les états de transition spéciaux, comme le wildcard `*` et `void`. Il montre aussi comment ces états sont utilisés pour les éléments qui entrent dans une vue ou en sortent.
Cette section explore également les déclencheurs d'animation multiples, les callbacks d'animation et les animations séquentielles à base de keyframes.

## États prédéfinis et correspondance wildcard {#predefined-states-and-wildcard-matching}

Dans Angular, les états de transition peuvent être définis explicitement avec la fonction [`state()`](api/animations/state), ou à l'aide des états prédéfinis `*` (wildcard) et `void`.

### État wildcard {#wildcard-state}

Un astérisque `*`, ou _wildcard_, correspond à n'importe quel état d'animation.
C'est utile pour définir des transitions qui s'appliquent quel que soit l'état de départ ou d'arrivée de l'élément HTML.

Par exemple, une transition `open => *` s'applique quand l'état de l'élément passe de open à n'importe quel autre état.

<img alt="expressions d'état wildcard" src="assets/images/guide/animations/wildcard-state-500.png">

Voici un autre exemple de code qui utilise l'état wildcard avec l'exemple précédent basé sur les états `open` et `closed`.
Au lieu de définir chaque paire de transitions d'un état à l'autre, toute transition vers `closed` dure 1 seconde, et toute transition vers `open` dure 0,5 seconde.

Cela permet d'ajouter de nouveaux états sans avoir à inclure des transitions distinctes pour chacun d'eux.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="trigger-wildcard1"/>

Utilisez la syntaxe à double flèche pour spécifier des transitions d'un état à l'autre dans les deux sens.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="trigger-wildcard2"/>

### Utiliser l'état wildcard avec plusieurs états de transition {#use-wildcard-state-with-multiple-transition-states}

Dans l'exemple du bouton à deux états, le wildcard n'est pas très utile, car il n'existe que deux états possibles, `open` et `closed`.
En général, utilisez les états wildcard quand un élément peut passer à plusieurs états potentiels.
Si le bouton peut passer de `open` à `closed` ou à un état comme `inProgress`, un état wildcard peut réduire la quantité de code nécessaire.

<img alt="état wildcard avec 3 états" src="assets/images/guide/animations/wildcard-3-states.png">

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="trigger-transition"/>

La transition `* => *` s'applique à tout changement entre deux états.

Les transitions sont mises en correspondance dans l'ordre où elles sont définies.
Vous pouvez donc appliquer d'autres transitions en plus de la transition `* => *`.
Par exemple, définissez des changements de style ou des animations qui s'appliquent uniquement à `open => closed`, puis utilisez `* => *` comme solution de repli pour les paires d'états qui ne sont pas explicitement traitées.

Pour cela, listez les transitions les plus spécifiques _avant_ `* => *`.

### Utiliser les wildcards avec des styles {#use-wildcards-with-styles}

Utilisez le wildcard `*` avec un style pour indiquer à l'animation d'utiliser la valeur actuelle de ce style, quelle qu'elle soit, et d'animer à partir de celle-ci.
Le wildcard est une valeur de repli utilisée si l'état animé n'est pas déclaré dans le déclencheur.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="transition4"/>

### État void {#void-state}

Utilisez l'état `void` pour configurer les transitions d'un élément qui entre dans une page ou en sort.
Consultez [Animer l'entrée et la sortie d'une vue](guide/legacy-animations/transition-and-triggers#aliases-enter-and-leave).

### Combiner les états wildcard et void {#combine-wildcard-and-void-states}

Combinez les états wildcard et void dans une transition pour déclencher des animations à l'entrée et à la sortie de la page :

- Une transition `* => void` s'applique quand l'élément quitte une vue, quel que soit l'état dans lequel il se trouvait avant de la quitter
- Une transition `void => *` s'applique quand l'élément entre dans une vue, quel que soit l'état qu'il prend en entrant
- L'état wildcard `*` correspond à _n'importe quel_ état, y compris `void`

## Animer l'entrée et la sortie d'une vue {#animate-entering-and-leaving-a-view}

Cette section montre comment animer des éléments qui entrent dans une page ou en sortent.

Ajoutez un nouveau comportement :

- Quand vous ajoutez un héros à la liste des héros, il semble arriver sur la page en volant depuis la gauche
- Quand vous retirez un héros de la liste, il semble s'envoler vers la droite

<docs-code header="hero-list-enter-leave.ts" path="adev/src/content/examples/animations/src/app/hero-list-enter-leave.ts" region="animationdef"/>

Dans le code précédent, vous avez appliqué l'état `void` lorsque l'élément HTML n'est pas attaché à une vue.

## Alias :enter et :leave {#aliases-enter-and-leave}

`:enter` et `:leave` sont des alias des transitions `void => *` et `* => void`.
Ces alias sont utilisés par plusieurs fonctions d'animation.

```ts {hideCopy}

transition ( ':enter', [ … ] ); // alias for void => _
transition ( ':leave', [ … ] ); // alias for _ => void

```

Il est plus difficile de cibler un élément qui entre dans une vue, car il n'est pas encore dans le DOM.
Utilisez les alias `:enter` et `:leave` pour cibler les éléments HTML insérés dans une vue ou retirés d'une vue.

### Utiliser `*ngIf` et `*ngFor` avec :enter et :leave {#use-ngif-and-ngfor-with-enter-and-leave}

La transition `:enter` s'exécute quand des vues `*ngIf` ou `*ngFor` sont placées sur la page, et `:leave` s'exécute quand ces vues sont retirées de la page.

IMPORTANT: Les comportements d'entrée et de sortie peuvent parfois prêter à confusion.
En règle générale, considérez que tout élément ajouté au DOM par Angular passe par la transition `:enter`. Seuls les éléments retirés directement du DOM par Angular passent par la transition `:leave`. Par exemple, la vue d'un élément est retirée du DOM parce que son parent est retiré du DOM.

Cet exemple comporte un déclencheur spécial pour l'animation d'entrée et de sortie, appelé `myInsertRemoveTrigger`.
Le template HTML contient le code suivant.

<docs-code header="insert-remove.html" path="adev/src/content/examples/animations/src/app/insert-remove.html" region="insert-remove"/>

Dans le fichier du composant, la transition `:enter` définit une opacité initiale de 0. Elle anime ensuite cette opacité jusqu'à 1 à mesure que l'élément est inséré dans la vue.

<docs-code header="insert-remove.ts" path="adev/src/content/examples/animations/src/app/insert-remove.ts" region="enter-leave-trigger"/>

Notez que cet exemple n'a pas besoin d'utiliser [`state()`](api/animations/state).

## Transitions :increment et :decrement {#transition-increment-and-decrement}

La fonction `transition()` accepte d'autres valeurs de sélecteur, `:increment` et `:decrement`.
Utilisez-les pour lancer une transition quand une valeur numérique a augmenté ou diminué.

HELPFUL: L'exemple suivant utilise les méthodes `query()` et `stagger()`.
Pour en savoir plus sur ces méthodes, consultez la page sur les [séquences complexes](guide/legacy-animations/complex-sequences).

<docs-code header="hero-list-page.ts" path="adev/src/content/examples/animations/src/app/hero-list-page.ts" region="increment"/>

## Valeurs booléennes dans les transitions {#boolean-values-in-transitions}

Si un déclencheur a pour valeur de binding un booléen, cette valeur peut être mise en correspondance à l'aide d'une expression `transition()` qui compare `true` et `false`, ou `1` et `0`.

<docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/open-close.2.html" region="trigger-boolean"/>

Dans l'extrait de code ci-dessus, le template HTML lie un élément `<div>` à un déclencheur nommé `openClose` avec une expression d'état `isOpen`, dont les valeurs possibles sont `true` et `false`.
Ce pattern est une alternative à la pratique qui consiste à créer deux états nommés comme `open` et `close`.

Dans les métadonnées de `@Component`, sous la propriété `animations:`, quand l'état s'évalue à `true`, la hauteur de l'élément HTML associé est un style wildcard, ou par défaut.
Dans ce cas, l'animation utilise la hauteur que l'élément avait déjà avant le début de l'animation.
Quand l'élément est `closed`, il est animé jusqu'à une hauteur de 0, ce qui le rend invisible.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.2.ts" region="trigger-boolean"/>

## Déclencheurs d'animation multiples {#multiple-animation-triggers}

Vous pouvez définir plusieurs déclencheurs d'animation pour un composant.
Attachez les déclencheurs d'animation à différents éléments : les relations parent-enfant entre ces éléments influencent la manière dont les animations s'exécutent et le moment où elles le font.

### Animations parent-enfant {#parent-child-animations}

Chaque fois qu'une animation est déclenchée dans Angular, l'animation parente a toujours la priorité et les animations enfants sont bloquées.
Pour qu'une animation enfant s'exécute, l'animation parente doit cibler par une query chacun des éléments qui contiennent des animations enfants. Elle laisse ensuite les animations s'exécuter à l'aide de la fonction [`animateChild()`](api/animations/animateChild).

#### Désactiver une animation sur un élément HTML {#disable-an-animation-on-an-html-element}

Un binding spécial de contrôle des animations, appelé `@.disabled`, peut être placé sur un élément HTML pour désactiver les animations sur cet élément ainsi que sur tous les éléments imbriqués.
Quand il vaut true, le binding `@.disabled` empêche le rendu de toutes les animations.

L'exemple de code suivant montre comment utiliser cette fonctionnalité.

<docs-code-multifile>
    <docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/open-close.4.html" region="toggle-animation"/>
    <docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.4.ts" region="toggle-animation" language="typescript"/>
</docs-code-multifile>

Quand le binding `@.disabled` vaut true, le déclencheur `@childAnimation` ne se lance pas.

Quand les animations d'un élément d'un template HTML sont désactivées à l'aide du host binding `@.disabled`, elles sont aussi désactivées sur tous les éléments internes.
Vous ne pouvez pas désactiver sélectivement plusieurs animations sur un même élément.<!-- vale off -->

Certaines animations enfants peuvent néanmoins s'exécuter sur un parent désactivé, de l'une des façons suivantes :

- Une animation parente peut utiliser la fonction [`query()`](api/animations/query) pour collecter les éléments internes situés dans des zones désactivées du template HTML.
Ces éléments peuvent quand même être animés.
<!-- vale on -->

- Une animation enfant peut être ciblée par une query du parent, puis animée ensuite avec la fonction `animateChild()`

#### Désactiver toutes les animations {#disable-all-animations}

Pour désactiver toutes les animations d'une application Angular, placez le host binding `@.disabled` sur le composant Angular de plus haut niveau.

<docs-code header="app.ts" path="adev/src/content/examples/animations/src/app/app.ts" region="toggle-app-animations"/>

HELPFUL: Désactiver les animations dans toute l'application est utile pendant les tests de bout en bout \(E2E\).

## Callbacks d'animation {#animation-callbacks}

La fonction d'animation `trigger()` émet des _callbacks_ quand elle démarre et quand elle se termine.
L'exemple suivant présente un composant qui contient un déclencheur `openClose`.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="events1"/>

Dans le template HTML, l'événement d'animation est renvoyé via `$event`, sous la forme `@triggerName.start` et `@triggerName.done`, où `triggerName` est le nom du déclencheur utilisé.
Dans cet exemple, le déclencheur `openClose` se présente ainsi.

<docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/open-close.3.html" region="callbacks"/>

Les callbacks d'animation peuvent par exemple servir à masquer la lenteur d'un appel d'API, comme une recherche en base de données.
Par exemple, un bouton **InProgress** peut être configuré pour avoir sa propre animation en boucle pendant que l'opération du système backend se termine.

Une autre animation peut être appelée quand l'animation en cours se termine.
Par exemple, le bouton passe de l'état `inProgress` à l'état `closed` quand l'appel d'API est terminé.

Une animation peut amener l'utilisateur final à _percevoir_ l'opération comme plus rapide, même quand elle ne l'est pas.

Les callbacks peuvent servir d'outil de débogage, par exemple avec `console.warn()` pour suivre la progression de l'application dans la console JavaScript des outils de développement du navigateur.
L'extrait de code suivant produit une sortie dans la console pour l'exemple d'origine, un bouton avec les deux états `open` et `closed`.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="events"/>

## Keyframes {#keyframes}

Pour créer une animation composée de plusieurs étapes exécutées en séquence, utilisez des _keyframes_.

La fonction `keyframe()` d'Angular permet plusieurs changements de style au sein d'un même segment temporel.
Par exemple, au lieu de s'estomper, le bouton pourrait changer de couleur plusieurs fois sur une même durée de 2 secondes.

<img alt="keyframes" src="assets/images/guide/animations/keyframes-500.png">

Le code de ce changement de couleur pourrait ressembler à ceci.

<docs-code header="status-slider.ts" path="adev/src/content/examples/animations/src/app/status-slider.ts" region="keyframes"/>

### Offset {#offset}

Les keyframes incluent un `offset` qui définit le moment de l'animation où chaque changement de style se produit.
Les offsets sont des mesures relatives allant de zéro à un, qui marquent le début et la fin de l'animation. Si vous en utilisez au moins un, ils doivent être appliqués à chacune des étapes de keyframes.

La définition des offsets des keyframes est facultative.
Si vous les omettez, des offsets régulièrement espacés sont attribués automatiquement.
Par exemple, trois keyframes sans offsets prédéfinis reçoivent les offsets 0, 0,5 et 1.
Spécifier un offset de 0,8 pour la transition du milieu dans l'exemple précédent pourrait ressembler à ceci.

<img alt="keyframes avec offset" src="assets/images/guide/animations/keyframes-offset-500.png">

Le code avec les offsets spécifiés serait le suivant.

<docs-code header="status-slider.ts" path="adev/src/content/examples/animations/src/app/status-slider.ts" region="keyframesWithOffsets"/>

Vous pouvez combiner des keyframes avec `duration`, `delay` et `easing` dans une même animation.

### Keyframes avec une pulsation {#keyframes-with-a-pulsation}

Utilisez des keyframes pour créer un effet de pulsation dans vos animations en définissant des styles à des offsets précis tout au long de l'animation.

Voici un exemple d'utilisation des keyframes pour créer un effet de pulsation :

- Les états d'origine `open` et `closed`, avec les changements d'origine de hauteur, de couleur et d'opacité, qui se produisent sur une durée de 1 seconde
- Une séquence de keyframes insérée au milieu, qui fait pulser le bouton de manière irrégulière au cours de cette même seconde

<img alt="keyframes avec pulsation irrégulière" src="assets/images/guide/animations/keyframes-pulsation.png">

L'extrait de code de cette animation pourrait ressembler à ceci.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.1.ts" region="trigger"/>

### Propriétés animables et unités {#animatable-properties-and-units}

Les animations Angular reposent sur les web animations, vous pouvez donc animer toute propriété que le navigateur considère comme animable.
Cela inclut les positions, les tailles, les transformations, les couleurs, les bordures, etc.
Le W3C tient à jour une liste des propriétés animables sur sa page [CSS Transitions](https://www.w3.org/TR/css-transitions-1).

Pour les propriétés à valeur numérique, définissez une unité en fournissant la valeur sous forme de chaîne, entre guillemets, avec le suffixe approprié :

- 50 pixels :
  `'50px'`

- Taille de police relative :
  `'3em'`

- Pourcentage :
  `'100%'`

Vous pouvez aussi fournir la valeur sous forme de nombre. Dans ce cas, Angular suppose une unité par défaut en pixels, soit `px`.
Exprimer 50 pixels par `50` revient à écrire `'50px'`.

HELPFUL: La chaîne `"50"`, en revanche, ne serait pas considérée comme valide.

### Calcul automatique des propriétés avec les wildcards {#automatic-property-calculation-with-wildcards}

Parfois, la valeur d'une propriété de style dimensionnelle n'est connue qu'à l'exécution.
Par exemple, les éléments ont souvent une largeur et une hauteur qui dépendent de leur contenu ou de la taille de l'écran.
Ces propriétés sont souvent difficiles à animer en CSS.

Dans ces cas, vous pouvez utiliser la valeur de propriété spéciale wildcard `*` dans `style()`. La valeur de cette propriété de style est calculée à l'exécution, puis injectée dans l'animation.

L'exemple suivant comporte un déclencheur appelé `shrinkOut`, utilisé quand un élément HTML quitte la page.
L'animation prend la hauteur qu'a l'élément avant de partir, et l'anime de cette hauteur jusqu'à zéro.

<docs-code header="hero-list-auto.ts" path="adev/src/content/examples/animations/src/app/hero-list-auto.ts" region="auto-calc"/>

### Résumé des keyframes {#keyframes-summary}

La fonction `keyframes()` d'Angular permet de spécifier plusieurs styles intermédiaires au sein d'une même transition. Un `offset` facultatif peut servir à définir le moment de l'animation où chaque changement de style doit se produire.

## En savoir plus sur les animations Angular {#more-on-angular-animations}

Les pages suivantes pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/legacy-animations" title="Introduction aux animations Angular"/>
  <docs-pill href="guide/legacy-animations/complex-sequences" title="Séquences d'animation complexes"/>
  <docs-pill href="guide/legacy-animations/reusable-animations" title="Animations réutilisables"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
  <docs-pill href="guide/animations/migration" title="Migrer vers les animations CSS natives"/>
</docs-pill-row>
