<!-- source: 1fd7803649645abab017a3707c6646cf78999749 -->
# Introduction aux animations Angular

IMPORTANT: Le package `@angular/animations` est désormais déprécié. L'équipe Angular recommande d'utiliser le CSS natif avec `animate.enter` et `animate.leave` pour les animations dans tout nouveau code. Pour en savoir plus, consultez le nouveau [guide des animations](guide/animations) d'entrée et de sortie. Consultez aussi [Abandonner le package Animations d'Angular](guide/animations/migration) pour savoir comment commencer à migrer vers des animations en CSS pur dans vos applications.

Une animation donne l'illusion du mouvement : les éléments HTML changent de style au fil du temps.
Des animations bien conçues peuvent rendre votre application plus intuitive et plus attrayante, mais elles ne sont pas seulement esthétiques.
Les animations peuvent améliorer votre application et l'expérience utilisateur de plusieurs façons :

- Sans animations, les transitions entre pages web peuvent sembler abruptes et brutales
- Le mouvement améliore grandement l'expérience utilisateur : les animations permettent aux utilisateurs de percevoir la réponse de l'application à leurs actions
- De bonnes animations attirent intuitivement l'attention de l'utilisateur là où c'est nécessaire

En général, les animations impliquent plusieurs _transformations_ de style au fil du temps.
Un élément HTML peut se déplacer, changer de couleur, grandir ou rétrécir, s'estomper ou glisser hors de la page.
Ces changements peuvent se produire simultanément ou successivement. Vous pouvez contrôler la durée de chaque transformation.

Le système d'animation d'Angular repose sur les fonctionnalités CSS, ce qui signifie que vous pouvez animer toute propriété que le navigateur considère comme animable.
Cela inclut les positions, les tailles, les transformations, les couleurs, les bordures, etc.
Le W3C tient à jour une liste des propriétés animables sur sa page [CSS Transitions](https://www.w3.org/TR/css-transitions-1).

## À propos de ce guide {#about-this-guide}

Ce guide présente les fonctionnalités de base des animations Angular pour vous permettre de commencer à ajouter des animations Angular à votre projet.

## Premiers pas {#getting-started}

Les principaux modules Angular pour les animations sont `@angular/animations` et `@angular/platform-browser`.

Pour commencer à ajouter des animations Angular à votre projet, importez les modules propres aux animations en plus des fonctionnalités Angular standard.

<docs-workflow>
<docs-step title="Activer le module d'animations {#enabling-the-animations-module}">
Importez `provideAnimationsAsync` depuis `@angular/platform-browser/animations/async` et ajoutez-le à la liste des providers dans l'appel à la fonction `bootstrapApplication`.

```ts {header: "Enabling Animations", linenums}
bootstrapApplication(AppComponent, {
  providers: [provideAnimationsAsync()],
});
```

<docs-callout important title="Si vous avez besoin d'animations immédiates dans votre application">
  Si une animation doit se produire immédiatement au chargement de votre application,
  vous devrez passer au module d'animations chargé de manière anticipée (eager). Importez plutôt `provideAnimations`
  depuis `@angular/platform-browser/animations`, et utilisez `provideAnimations` **à la place de**
  `provideAnimationsAsync` dans l'appel à la fonction `bootstrapApplication`.
</docs-callout>

Pour les applications basées sur `NgModule`, importez `BrowserAnimationsModule`, qui apporte les fonctionnalités d'animation au module racine de votre application Angular.

<docs-code header="app.module.ts" path="adev/src/content/examples/animations/src/app/app.module.1.ts"/>
</docs-step>
<docs-step title="Importer les fonctions d'animation dans les fichiers de composant {#importing-animation-functions-into-component-files}">
Si vous prévoyez d'utiliser des fonctions d'animation spécifiques dans des fichiers de composant, importez ces fonctions depuis `@angular/animations`.

<docs-code header="app.ts" path="adev/src/content/examples/animations/src/app/app.ts" region="imports"/>

Consultez toutes les [fonctions d'animation disponibles](guide/legacy-animations#animations-api-summary) à la fin de ce guide.

</docs-step>
<docs-step title="Ajouter la propriété de métadonnées animations {#adding-the-animation-metadata-property}">
Dans le fichier du composant, ajoutez une propriété de métadonnées appelée `animations:` dans le décorateur `@Component()`.
Placez le déclencheur qui définit une animation dans la propriété de métadonnées `animations`.

<docs-code header="app.ts" path="adev/src/content/examples/animations/src/app/app.ts" region="decorator"/>
</docs-step>
</docs-workflow>

## Animer une transition {#animating-a-transition}

Animons une transition qui fait passer un seul élément HTML d'un état à un autre.
Par exemple, vous pouvez indiquer qu'un bouton affiche **Open** ou **Closed** selon la dernière action de l'utilisateur.
Quand le bouton est dans l'état `open`, il est visible et jaune.
Quand il est dans l'état `closed`, il est translucide et bleu.

En HTML, ces attributs sont définis à l'aide de styles CSS ordinaires comme la couleur et l'opacité.
Dans Angular, utilisez la fonction `style()` pour spécifier un ensemble de styles CSS à utiliser avec les animations.
Regroupez un ensemble de styles dans un état d'animation et donnez un nom à cet état, comme `open` ou `closed`.

HELPFUL: Créons un nouveau composant `open-close` à animer avec des transitions simples.

Exécutez la commande suivante dans le terminal pour générer le composant :

```shell
ng g component open-close
```

Cela crée le composant dans `src/app/open-close.ts`.

### État et styles d'animation {#animation-state-and-styles}

Utilisez la fonction [`state()`](api/animations/state) d'Angular pour définir les différents états à atteindre à la fin de chaque transition.
Cette fonction prend deux arguments :
un nom unique comme `open` ou `closed` et une fonction `style()`.

Utilisez la fonction `style()` pour définir un ensemble de styles à associer à un nom d'état donné.
Vous devez utiliser le _camelCase_ pour les attributs de style qui contiennent des tirets, comme `backgroundColor`, ou les entourer de guillemets, comme `'background-color'`.

Voyons comment la fonction [`state()`](api/animations/state) d'Angular fonctionne avec la fonction `style⁣­(⁠)` pour définir des attributs de style CSS.
Dans cet extrait de code, plusieurs attributs de style sont définis en même temps pour l'état.
Dans l'état `open`, le bouton a une hauteur de 200 pixels, une opacité de 1 et une couleur de fond jaune.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="state1"/>

Dans l'état `closed` qui suit, le bouton a une hauteur de 100 pixels, une opacité de 0,8 et une couleur de fond bleue.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="state2"/>

### Transitions et timing {#transitions-and-timing}

Dans Angular, vous pouvez définir plusieurs styles sans aucune animation.
Cependant, sans autre ajustement, le bouton se transforme instantanément, sans fondu, sans rétrécissement ni autre indicateur visible qu'un changement est en cours.

Pour rendre le changement moins abrupt, vous devez définir une _transition_ d'animation qui spécifie les changements se produisant entre un état et un autre sur une certaine durée.
La fonction `transition()` accepte deux arguments :
le premier accepte une expression qui définit le sens entre deux états de transition, et le second accepte une ou plusieurs étapes `animate()`.

Utilisez la fonction `animate()` pour définir la durée, le délai et l'easing d'une transition, et pour désigner la fonction de style qui définit les styles pendant le déroulement des transitions.
Utilisez la fonction `animate()` pour définir la fonction `keyframes()` des animations en plusieurs étapes.
Ces définitions se placent dans le second argument de la fonction `animate()`.

#### Métadonnées d'animation : durée, délai et easing {#animation-metadata-duration-delay-and-easing}

La fonction `animate()` \(second argument de la fonction de transition\) accepte les paramètres d'entrée `timings` et `styles`.

Le paramètre `timings` prend soit un nombre, soit une chaîne composée de trois parties.

```ts
animate(duration);
```

ou

```ts
animate('duration delay easing');
```

La première partie, `duration`, est obligatoire.
La durée peut s'exprimer en millisecondes sous forme de nombre sans guillemets, ou en secondes avec des guillemets et une unité de temps.
Par exemple, une durée d'un dixième de seconde peut s'exprimer ainsi :

- Sous forme de nombre simple, en millisecondes :
  `100`

- Dans une chaîne, en millisecondes :
  `'100ms'`

- Dans une chaîne, en secondes :
  `'0.1s'`

Le deuxième argument, `delay`, a la même syntaxe que `duration`.
Par exemple :

- Attendre 100 ms puis s'exécuter pendant 200 ms : `'0.2s 100ms'`

Le troisième argument, `easing`, contrôle la façon dont l'animation [accélère et décélère](https://easings.net) pendant son exécution.
Par exemple, `ease-in` fait démarrer l'animation lentement, puis elle prend de la vitesse au fur et à mesure.

- Attendre 100 ms, s'exécuter pendant 200 ms.
  Utiliser une courbe de décélération pour démarrer vite et ralentir progressivement jusqu'à un point de repos :
  `'0.2s 100ms ease-out'`

- S'exécuter pendant 200 ms, sans délai.
  Utiliser une courbe standard pour démarrer lentement, accélérer au milieu, puis ralentir progressivement à la fin :
  `'0.2s ease-in-out'`

- Démarrer immédiatement, s'exécuter pendant 200 ms.
  Utiliser une courbe d'accélération pour démarrer lentement et finir à pleine vitesse :
  `'0.2s ease-in'`

HELPFUL: Consultez la page du site Material Design consacrée aux [courbes d'easing naturelles](https://material.io/design/motion/speed.html#easing) pour des informations générales sur les courbes d'easing.

Cet exemple fournit une transition d'état de `open` à `closed` avec une transition d'une seconde entre les états.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="transition1"/>

Dans l'extrait de code précédent, l'opérateur `=>` indique des transitions unidirectionnelles, et `<=>` est bidirectionnel.
Dans la transition, `animate()` spécifie la durée de la transition.
Ici, le changement d'état de `open` à `closed` dure 1 seconde, exprimée ici par `1s`.

Cet exemple ajoute une transition d'état de l'état `closed` vers l'état `open` avec un arc d'animation de transition de 0,5 seconde.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="transition2"/>

HELPFUL: Quelques remarques supplémentaires sur l'utilisation des styles dans les fonctions [`state`](api/animations/state) et `transition`.

- Utilisez [`state()`](api/animations/state) pour définir les styles appliqués à la fin de chaque transition ; ils persistent une fois l'animation terminée
- Utilisez `transition()` pour définir des styles intermédiaires, qui créent l'illusion du mouvement pendant l'animation
- Quand les animations sont désactivées, les styles de `transition()` peuvent être ignorés, mais pas ceux de [`state()`](api/animations/state)
- Incluez plusieurs paires d'états dans le même argument de `transition()` :

  ```ts
  transition('on => off, off => void');
  ```

### Déclencher l'animation {#triggering-the-animation}

Une animation nécessite un _déclencheur_ pour savoir quand démarrer.
La fonction `trigger()` regroupe les états et les transitions, et donne un nom à l'animation, afin que vous puissiez l'attacher à l'élément déclencheur dans le template HTML.

La fonction `trigger()` décrit le nom de la propriété dont il faut surveiller les changements.
Quand un changement se produit, le déclencheur lance les actions incluses dans sa définition.
Ces actions peuvent être des transitions ou d'autres fonctions, comme nous le verrons plus loin.

Dans cet exemple, nous nommons le déclencheur `openClose` et nous l'attachons à l'élément `button`.
Le déclencheur décrit les états ouvert et fermé, ainsi que le timing des deux transitions.

HELPFUL: Dans chaque appel à la fonction `trigger()`, un élément ne peut être que dans un seul état à un instant donné.
Il est cependant possible que plusieurs déclencheurs soient actifs en même temps.

### Définir des animations et les attacher au template HTML {#defining-animations-and-attaching-them-to-the-html-template}

Les animations sont définies dans les métadonnées du composant qui contrôle l'élément HTML à animer.
Placez le code qui définit vos animations sous la propriété `animations:` dans le décorateur `@Component()`.

<docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="component"/>

Une fois que vous avez défini un déclencheur d'animation pour un composant, attachez-le à un élément du template de ce composant en plaçant le nom du déclencheur entre crochets, précédé du symbole `@`.
Vous pouvez ensuite lier le déclencheur à une expression de template avec la syntaxe standard du property binding d'Angular, comme ci-dessous, où `triggerName` est le nom du déclencheur et où `expression` s'évalue en un état d'animation défini.

```angular-html
<div [@triggerName]="expression">…</div>
```

L'animation est exécutée, ou déclenchée, quand la valeur de l'expression passe à un nouvel état.

L'extrait de code suivant lie le déclencheur à la valeur de la propriété `isOpen`.

<docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/open-close.1.html" region="trigger"/>

Dans cet exemple, quand l'expression `isOpen` s'évalue en un état défini, `open` ou `closed`, elle notifie le déclencheur `openClose` d'un changement d'état.
C'est ensuite au code de `openClose` de gérer le changement d'état et de lancer une animation de changement d'état.

Pour les éléments qui entrent dans une page ou en sortent \(insérés dans le DOM ou retirés du DOM\), vous pouvez rendre les animations conditionnelles.
Par exemple, utilisez `*ngIf` avec le déclencheur d'animation dans le template HTML.

HELPFUL: Dans le fichier du composant, définissez le déclencheur qui définit les animations comme valeur de la propriété `animations:` du décorateur `@Component()`.

Dans le fichier de template HTML, utilisez le nom du déclencheur pour attacher les animations définies à l'élément HTML à animer.

### Revue du code {#code-review}

Voici les fichiers de code présentés dans l'exemple de transition.

<docs-code-multifile>
    <docs-code header="open-close.ts" path="adev/src/content/examples/animations/src/app/open-close.ts" region="component"/>
    <docs-code header="open-close.html" path="adev/src/content/examples/animations/src/app/open-close.1.html" region="trigger"/>
    <docs-code header="open-close.css" path="adev/src/content/examples/animations/src/app/open-close.css"/>
</docs-code-multifile>

### Résumé {#summary}

Vous avez appris à ajouter une animation à une transition entre deux états, en utilisant `style()` et [`state()`](api/animations/state) ainsi que `animate()` pour le timing.

Découvrez des fonctionnalités plus avancées des animations Angular dans la section Animation, en commençant par les techniques avancées présentées dans [transitions et déclencheurs](guide/legacy-animations/transition-and-triggers).

## Résumé de l'API d'animations {#animations-api-summary}

L'API fonctionnelle fournie par le module `@angular/animations` offre un langage dédié \(DSL\) pour créer et contrôler des animations dans les applications Angular.
Consultez la [référence de l'API](api?package=angular_animations&status=8) pour la liste complète et les détails de syntaxe des fonctions principales et des structures de données associées.

| Nom de la fonction                | Rôle                                                                                                                                                                                                        |
| :-------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `trigger()`                       | Lance l'animation et sert de conteneur pour tous les autres appels de fonctions d'animation. Le template HTML se lie à `triggerName`. Utilisez le premier argument pour déclarer un nom de déclencheur unique. Utilise la syntaxe de tableau. |
| `style()`                         | Définit un ou plusieurs styles CSS à utiliser dans les animations. Contrôle l'apparence visuelle des éléments HTML pendant les animations. Utilise la syntaxe d'objet. |
| [`state()`](api/animations/state) | Crée un ensemble nommé de styles CSS à appliquer lorsqu'une transition vers un état donné aboutit. L'état peut ensuite être référencé par son nom dans d'autres fonctions d'animation. |
| `animate()`                       | Spécifie les informations de timing d'une transition. Valeurs optionnelles pour `delay` et `easing`. Peut contenir des appels à `style()`. |
| `transition()`                    | Définit la séquence d'animation entre deux états nommés. Utilise la syntaxe de tableau. |
| `keyframes()`                     | Permet un changement séquentiel entre des styles sur un intervalle de temps donné. À utiliser dans `animate()`. Peut inclure plusieurs appels à `style()` dans chaque `keyframe()`. Utilise la syntaxe de tableau. |
| [`group()`](api/animations/group) | Spécifie un groupe d'étapes d'animation \(_animations internes_\) à exécuter en parallèle. L'animation ne continue qu'une fois toutes les étapes d'animation internes terminées. S'utilise dans `sequence()` ou `transition()`. |
| `query()`                         | Trouve un ou plusieurs éléments HTML internes dans l'élément courant. |
| `sequence()`                      | Spécifie une liste d'étapes d'animation exécutées séquentiellement, l'une après l'autre. |
| `stagger()`                       | Échelonne l'heure de début des animations de plusieurs éléments. |
| `animation()`                     | Produit une animation réutilisable qui peut être invoquée depuis un autre endroit. S'utilise avec `useAnimation()`. |
| `useAnimation()`                  | Active une animation réutilisable. S'utilise avec `animation()`. |
| `animateChild()`                  | Permet aux animations des composants enfants de s'exécuter dans la même plage de temps que celles du parent. |

</table>

## En savoir plus sur les animations Angular {#more-on-angular-animations}

HELPFUL: Regardez cette [présentation](https://www.youtube.com/watch?v=rnTK9meY5us), donnée lors de la conférence AngularConnect en novembre 2017, ainsi que le [code source](https://github.com/matsko/animationsftw.in) qui l'accompagne.

Les pages suivantes pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/legacy-animations/transition-and-triggers" title="Transitions et déclencheurs"/>
  <docs-pill href="guide/legacy-animations/complex-sequences" title="Séquences d'animation complexes"/>
  <docs-pill href="guide/legacy-animations/reusable-animations" title="Animations réutilisables"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
  <docs-pill href="guide/animations/migration" title="Migrer vers les animations CSS natives"/>
</docs-pill-row>
