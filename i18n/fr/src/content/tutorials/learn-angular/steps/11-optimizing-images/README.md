<!-- source: 205ab63f3148e5f72d89f8ab7c1076e3a4b748f8 -->
# Optimiser les images

Les images occupent une grande place dans de nombreuses applications et peuvent contribuer fortement aux problèmes de performances, notamment à de faibles scores [Core Web Vitals](https://web.dev/explore/learn-core-web-vitals).

L'optimisation des images peut être un sujet complexe, mais Angular en gère l'essentiel pour vous avec la directive `NgOptimizedImage`.

NOTE: Pour en savoir plus, consultez [l'optimisation des images avec NgOptimizedImage dans le guide approfondi](/guide/image-optimization).

Dans cette activité, vous allez apprendre à utiliser `NgOptimizedImage` pour que vos images soient chargées efficacement.

<hr>

<docs-workflow>

<docs-step title="Importer la directive NgOptimizedImage {#import-the-ngoptimizedimage-directive}">

Pour tirer parti de la directive `NgOptimizedImage`, importez-la d'abord depuis la library `@angular/common` et ajoutez-la au tableau `imports` du composant.

```ts
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  ...
})
```

</docs-step>

<docs-step title="Remplacer l'attribut src par ngSrc {#update-the-src-attribute-to-be-ngsrc}">

Pour activer la directive `NgOptimizedImage`, remplacez l'attribut `src` par `ngSrc`. Cela s'applique aussi bien aux sources d'images statiques (c'est-à-dire `src`) qu'aux sources d'images dynamiques (c'est-à-dire `[src]`).

```angular-ts {highlight:[[7],[11]]}
import { NgOptimizedImage } from '@angular/common';

@Component({
template: `     ...
    <li>
      Static Image:
      <img ngSrc="/logo.svg" alt="Angular logo" width="32" height="32" />
    </li>
    <li>
      Dynamic Image:
      <img [ngSrc]="logoUrl" [alt]="logoAlt" width="32" height="32" />
    </li>
    ...
  `,
imports: [NgOptimizedImage],
})
```

</docs-step>

<docs-step title="Ajouter les attributs width et height {#add-width-and-height-attributes}">

Notez que, dans l'exemple de code ci-dessus, chaque image possède à la fois un attribut `width` et un attribut `height`. Pour éviter le [décalage de mise en page](https://web.dev/articles/cls), la directive `NgOptimizedImage` exige ces deux attributs de taille sur chaque image.

Lorsque vous ne pouvez pas ou ne voulez pas spécifier de `height` et de `width` statiques pour les images, vous pouvez utiliser [l'attribut `fill`](/guide/image-optimization#using-fill-mode) pour indiquer à l'image de se comporter comme une « image d'arrière-plan », en remplissant l'élément qui la contient :

```angular-html
// Container div has 'position: "relative"'
<div class="image-container">
  <img ngSrc="www.example.com/image.png" fill />
</div>
```

NOTE: Pour que l'image `fill` s'affiche correctement, son élément parent doit avoir le style `position: "relative"`, `position: "fixed"` ou `position: "absolute"`.

</docs-step>

<docs-step title="Prioriser les images importantes {#prioritize-important-images}">

L'une des optimisations les plus importantes pour les performances de chargement consiste à donner la priorité à toute image susceptible d'être l'[« élément LCP »](https://web.dev/articles/optimize-lcp), c'est-à-dire le plus grand élément graphique affiché à l'écran au chargement de la page. Pour optimiser vos temps de chargement, pensez à ajouter l'attribut `priority` à votre « image hero » ou à toute autre image qui pourrait selon vous être un élément LCP.

```ts
<img ngSrc="www.example.com/image.png" height="600" width="800" priority />
```

</docs-step>

<docs-step title="Facultatif : utiliser un loader d'images {#optional-use-an-image-loader}">

`NgOptimizedImage` vous permet de spécifier un [loader d'images](guide/image-optimization#configuring-an-image-loader-for-ngoptimizedimage), qui indique à la directive comment formater les URL de vos images. Un loader vous permet de définir vos images avec des URL courtes et relatives :

```ts
providers: [provideImgixLoader('https://my.base.url/')],
```

L'URL finale sera 'https://my.base.url/image.png'

```angular-html
<img ngSrc="image.png" height="600" width="800" />
```

Les loaders d'images ne servent pas qu'à simplifier l'écriture : ils vous permettent d'exploiter toutes les capacités de `NgOptimizedImage`. Pour en savoir plus sur ces optimisations et sur les loaders intégrés pour les CDN les plus courants, consultez [cette section](guide/image-optimization#configuring-an-image-loader-for-ngoptimizedimage).

</docs-step>

</docs-workflow>

En ajoutant cette directive à votre workflow, vos images se chargent désormais selon les bonnes pratiques, avec l'aide d'Angular 🎉

Pour en savoir plus, consultez la [documentation de `NgOptimizedImage`](guide/image-optimization). Continuez votre excellent travail ; découvrons maintenant le routing.
