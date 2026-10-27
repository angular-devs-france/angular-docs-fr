<!-- source: 9c774c0638d53ec832fad7e566936db342026b6d -->
# Utiliser les API du DOM

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Lisez-le d'abord si vous découvrez Angular.

Angular gère pour vous la plupart des créations, mises à jour et suppressions dans le DOM. Il peut toutefois arriver, rarement, que vous ayez besoin
d'interagir directement avec le DOM d'un composant. Les composants peuvent injecter ElementRef pour obtenir une référence à
le host element du composant :

```ts
@Component(/* ... */)
export class ProfilePhoto {
  constructor() {
    const elementRef = inject(ElementRef);
    console.log(elementRef.nativeElement);
  }
}
```

La propriété `nativeElement` référence l'instance
d'[Element](https://developer.mozilla.org/docs/Web/API/Element) du host element.

Vous pouvez utiliser les fonctions `afterEveryRender` et `afterNextRender` d'Angular pour enregistrer un **callback
de rendu** qui s'exécute quand Angular a fini de rendre la page.

```ts
@Component(/* ... */)
export class ProfilePhoto {
  constructor() {
    const elementRef = inject(ElementRef);
    afterEveryRender(() => {
      // Focus the first input element in this component.
      elementRef.nativeElement.querySelector('input')?.focus();
    });
  }
}
```

`afterEveryRender` et `afterNextRender` doivent être appelées dans un _contexte d'injection_, généralement le
constructeur d'un composant.

**Évitez la manipulation directe du DOM chaque fois que c'est possible.** Préférez toujours exprimer la structure de votre DOM
dans les templates des composants et mettre à jour ce DOM avec des bindings.

**Les callbacks de rendu ne s'exécutent jamais pendant le rendu côté serveur ni pendant le prérendu au moment du build.**

**Ne manipulez jamais directement le DOM dans les autres lifecycle hooks d'Angular**. Angular ne
garantit pas que le DOM d'un composant soit entièrement rendu à un autre moment que dans les callbacks de rendu.
De plus, lire ou modifier le DOM pendant d'autres lifecycle hooks peut dégrader les performances de la page
en
provoquant du [layout thrashing](https://web.dev/avoid-large-complex-layouts-and-layout-thrashing).

## Utiliser le renderer d'un composant {#using-a-components-renderer}

Les composants peuvent injecter une instance de `Renderer2` pour effectuer certaines manipulations du DOM liées
à d'autres fonctionnalités d'Angular.

Tous les éléments du DOM créés par le `Renderer2` d'un composant participent à
l'[encapsulation des styles](guide/components/styling#style-scoping) de ce composant.

Certaines API de `Renderer2` s'intègrent aussi au système d'animation d'Angular. Vous pouvez utiliser la méthode `setProperty`
pour mettre à jour des propriétés d'animation synthétiques et la méthode `listen` pour ajouter des event listeners pour
des événements d'animation synthétiques. Consultez le guide [Animations](guide/animations) pour plus de détails.

En dehors de ces deux cas d'usage restreints, il n'y a aucune différence entre l'utilisation de `Renderer2` et celle des API
natives du DOM. Les API de `Renderer2` ne prennent pas en charge la manipulation du DOM dans les contextes de rendu côté serveur ni de
prérendu au moment du build.

## Quand utiliser les API du DOM {#when-to-use-dom-apis}

Même si Angular gère la plupart des aspects du rendu, certains comportements peuvent encore nécessiter les API du DOM. Voici
quelques cas d'usage courants :

- Gérer le focus des éléments
- Mesurer la géométrie des éléments, par exemple avec `getBoundingClientRect`
- Lire le contenu textuel d'un élément
- Mettre en place des observateurs natifs tels
  que [`MutationObserver`](https://developer.mozilla.org/docs/Web/API/MutationObserver),
  [`ResizeObserver`](https://developer.mozilla.org/docs/Web/API/ResizeObserver) ou
  [`IntersectionObserver`](https://developer.mozilla.org/docs/Web/API/Intersection_Observer_API).

Évitez d'insérer, de supprimer et de modifier des éléments du DOM. En particulier, **ne définissez jamais directement la propriété
`innerHTML` d'un élément**, ce qui peut rendre votre application vulnérable
aux [failles de cross-site scripting (XSS)](https://developer.mozilla.org/docs/Glossary/Cross-site_scripting).
Les bindings de template d'Angular, y compris les bindings sur `innerHTML`, incluent des garde-fous qui aident à
se protéger contre les attaques XSS. Consultez le [guide de sécurité](best-practices/security) pour plus de détails.
