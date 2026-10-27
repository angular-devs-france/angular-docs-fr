<!-- source: 5993a2fecf48c396ea6ebb13bec378fa35891acb -->
# Animer vos applications avec `animate.enter` et `animate.leave`

Des animations bien conçues peuvent rendre votre application plus intuitive et plus attrayante, mais elles ne sont pas que cosmétiques.
Les animations peuvent améliorer votre application et l'expérience utilisateur de plusieurs façons :

- Sans animations, les transitions entre pages web peuvent paraître abruptes et brutales
- Le mouvement améliore grandement l'expérience utilisateur : les animations permettent aux utilisateurs de percevoir la réponse de l'application à leurs actions
- De bonnes animations peuvent guider en douceur l'attention de l'utilisateur tout au long d'un parcours

Angular fournit `animate.enter` et `animate.leave` pour animer les éléments de votre application. Ces deux fonctionnalités appliquent des classes CSS d'entrée et de sortie au bon moment, ou appellent des fonctions pour appliquer des animations issues de libraries tierces. `animate.enter` et `animate.leave` ne sont pas des directives. Ce sont des API spéciales prises en charge directement par le compilateur Angular. Elles peuvent être utilisées directement sur des éléments, et aussi en tant que host binding.

## `animate.enter` {#animateenter}

Vous pouvez utiliser `animate.enter` pour animer des éléments lorsqu'ils _entrent_ dans le DOM. Vous pouvez définir des animations d'entrée avec des classes CSS utilisant des transitions ou des animations keyframes.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/enter.ts">
    <docs-code header="enter.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter.ts" />
    <docs-code header="enter.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter.html" />
    <docs-code header="enter.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter.css"/>
</docs-code-multifile>

Quand l'animation se termine, Angular retire du DOM la ou les classes que vous avez indiquées dans `animate.enter`. Les classes d'animation ne sont présentes que pendant que l'animation est active.

NOTE: Quand vous utilisez plusieurs animations keyframes ou propriétés de transition sur un élément, Angular ne retire toutes les classes qu'_après_ la fin de l'animation la plus longue.

Vous pouvez utiliser `animate.enter` avec toutes les autres fonctionnalités d'Angular, comme le control flow ou les expressions dynamiques. `animate.enter` accepte soit une chaîne de classes unique (plusieurs classes séparées par des espaces), soit un tableau de chaînes de classes.

Une remarque rapide sur l'utilisation des transitions CSS : si vous choisissez d'utiliser des transitions plutôt que des animations keyframes, les classes ajoutées à l'élément avec `animate.enter` représentent l'état _vers_ lequel la transition anime. Le CSS de base de votre élément correspond à son apparence quand aucune animation n'est en cours, ce qui est probablement proche de l'état final de la transition CSS. Vous devez donc tout de même l'associer à `@starting-style` pour disposer d'un état _de départ_ approprié afin que votre transition fonctionne.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/enter-binding.ts">
    <docs-code header="enter-binding.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter-binding.ts" />
    <docs-code header="enter-binding.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter-binding.html" />
    <docs-code header="enter-binding.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/enter-binding.css"/>
</docs-code-multifile>

## `animate.leave` {#animateleave}

Vous pouvez utiliser `animate.leave` pour animer des éléments lorsqu'ils _quittent_ le DOM. Vous pouvez définir des animations de sortie avec des classes CSS utilisant des transformations ou des animations keyframes.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/leave.ts">
    <docs-code header="leave.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave.ts" />
    <docs-code header="leave.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave.html" />
    <docs-code header="leave.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave.css"/>
</docs-code-multifile>

Quand l'animation se termine, Angular retire automatiquement l'élément animé du DOM.

NOTE: Quand vous utilisez plusieurs animations keyframes ou propriétés de transition sur un élément, Angular attend la fin de la plus longue de ces animations avant de retirer l'élément.

`animate.leave` peut aussi être utilisé avec des signals et d'autres bindings. Vous pouvez utiliser `animate.leave` avec une seule classe ou plusieurs classes. Indiquez-les soit sous forme de chaîne simple séparée par des espaces, soit sous forme de tableau de chaînes.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-binding.ts">
    <docs-code header="leave-binding.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-binding.ts" />
    <docs-code header="leave-binding.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-binding.html" />
    <docs-code header="leave-binding.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-binding.css"/>
</docs-code-multifile>

### Ordre de retrait des éléments {#element-removal-order}

L'exécution des animations `animate.leave` et le moment où une animation se produit comportent quelques subtilités. `animate.leave` fonctionne s'il est placé sur l'élément en cours de retrait ; et si `animate.leave` est placé sur un élément _descendant_ de l'élément en cours de retrait _au sein du template d'un même composant_, ces animations `animate.leave` enfants se produisent _avant_ que le nœud parent ne soit retiré du DOM. Vous pouvez ainsi animer la disparition des éléments enfants en toute confiance, sans que le nœud parent ne disparaisse prématurément.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-parent.ts">
    <docs-code header="leave-parent.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-parent.ts" />
    <docs-code header="leave-parent.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-parent.html" />
    <docs-code header="leave-parent.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-parent.css"/>
</docs-code-multifile>

IMPORTANT: Les animations enfants ne se déclenchent que pour les éléments situés dans le template d'un même composant. Si un élément en cours de retrait contient des composants enfants, les animations `animate.leave` définies dans les templates de ces composants enfants ne s'exécuteront **pas** avant le retrait du parent. Pour animer un composant enfant lors de son retrait, appliquez plutôt `animate.leave` directement sur le host element du composant enfant dans le template parent, ou gérez par programmation le déclenchement de l'animation dans le composant enfant et retardez le retrait du parent jusqu'à la fin de cette animation.

## Event bindings, fonctions et libraries tierces {#event-bindings-functions-and-third-party-libraries}

`animate.enter` et `animate.leave` prennent tous deux en charge la syntaxe d'event binding, qui permet des appels de fonction. Vous pouvez utiliser cette syntaxe pour appeler une fonction dans le code de votre composant ou utiliser des libraries d'animation tierces, comme [GSAP](https://gsap.com/), [anime.js](https://animejs.com/) ou toute autre library d'animation JavaScript.

<docs-code-multifile preview path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-event.ts">
    <docs-code header="leave-event.ts" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-event.ts" />
    <docs-code header="leave-event.html" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-event.html" />
    <docs-code header="leave-event.css" path="adev/src/content/examples/animations/src/app/enter-and-leave/leave-event.css"/>
</docs-code-multifile>

L'objet `$event` est de type `AnimationCallbackEvent`. Il contient l'élément en tant que `target` et fournit une fonction `animationComplete()` pour notifier le framework quand l'animation se termine.

IMPORTANT: Vous **devez** appeler la fonction `animationComplete()` quand vous utilisez `animate.leave` pour qu'Angular retire l'élément.

Si vous n'appelez pas `animationComplete()` quand vous utilisez `animate.leave`, Angular appelle automatiquement la fonction après un délai de quatre secondes. Vous pouvez configurer la durée de ce délai en fournissant le jeton `MAX_ANIMATION_TIMEOUT` en millisecondes.

```typescript
  { provide: MAX_ANIMATION_TIMEOUT, useValue: 6000 }
```

## Compatibilité avec les anciennes animations Angular {#compatibility-with-legacy-angular-animations}

Vous ne pouvez pas utiliser les anciennes animations en même temps que `animate.enter` et `animate.leave` dans un même composant. Cela entraînerait des classes d'entrée qui restent sur l'élément ou des nœuds sortants qui ne sont pas retirés. En dehors de ce cas, vous pouvez tout à fait utiliser à la fois les anciennes animations et les nouvelles animations `animate.enter` et `animate.leave` dans une même _application_. La seule réserve concerne la projection de contenu. Si vous projetez du contenu d'un composant utilisant les anciennes animations dans un autre composant utilisant `animate.enter` ou `animate.leave`, ou inversement, le comportement sera le même que s'ils étaient utilisés ensemble dans le même composant. Ce cas n'est pas pris en charge.

## Tests {#testing}

TestBed fournit une prise en charge intégrée pour activer ou désactiver les animations dans votre environnement de test. Les animations CSS nécessitent un navigateur pour s'exécuter, et beaucoup de leurs API ne sont pas disponibles dans un environnement de test. Par défaut, TestBed désactive les animations pour vous dans vos environnements de test.

Si vous voulez vérifier que les animations s'exécutent bien dans un test en navigateur, par exemple un test de bout en bout, vous pouvez configurer TestBed pour activer les animations en indiquant `animationsEnabled: true` dans votre configuration de test.

```typescript
TestBed.configureTestingModule({animationsEnabled: true});
```

Les animations de votre environnement de test se comporteront alors normalement.

NOTE: Certains environnements de test n'émettent pas les événements d'animation comme `animationstart`, `animationend` et leurs équivalents pour les transitions.

## En savoir plus sur les animations Angular {#more-on-angular-animations}

Ceci pourrait aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="guide/animations/css" title="Animations complexes avec CSS"/>
  <docs-pill href="guide/routing/route-transition-animations" title="Animations de transition de route"/>
</docs-pill-row>
