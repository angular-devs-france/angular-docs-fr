<!-- source: fdb16fa9a56a1c2e1a52d6765d247b0f2a214fcc -->
# Premiers pas avec NgOptimizedImage

La directive `NgOptimizedImage` facilite l'adoption des bonnes pratiques de performance pour le chargement des images.

La directive garantit que le chargement de l'image [Largest Contentful Paint (LCP)](https://web.dev/lcp) est prioritaire en :

- Définissant automatiquement l'attribut `fetchpriority` sur la balise `<img>`
- Chargeant par défaut les autres images en lazy loading
- Générant automatiquement une balise link de preconnect dans le head du document
- Générant automatiquement un attribut `srcset`
- Générant un [preload hint](https://developer.mozilla.org/docs/Web/HTML/Link_types/preload) si l'application utilise le SSR

En plus d'optimiser le chargement de l'image LCP, `NgOptimizedImage` impose un certain nombre de bonnes pratiques pour les images, comme :

- Utiliser des [URL de CDN d'images pour appliquer des optimisations d'image](https://web.dev/image-cdns/#how-image-cdns-use-urls-to-indicate-optimization-options)
- Empêcher le décalage de mise en page en exigeant `width` et `height`
- Avertir si `width` ou `height` ont été définis de manière incorrecte
- Avertir si l'image sera visuellement déformée lors du rendu

Si vous utilisez une image d'arrière-plan en CSS, [commencez ici](#how-to-migrate-your-background-image).

NOTE: Bien que la directive `NgOptimizedImage` soit devenue une fonctionnalité stable dans Angular 15, elle a été rétroportée et est également disponible comme fonctionnalité stable dans les versions 13.4.0 et 14.3.0.

## Premiers pas {#getting-started}

<docs-workflow>
<docs-step title="Importer la directive `NgOptimizedImage` {#import-ngoptimizedimage-directive}">
Importez la directive `NgOptimizedImage` depuis `@angular/common` :

```ts
import {NgOptimizedImage} from '@angular/common';
```

puis ajoutez-la au tableau `imports` d'un composant standalone ou d'un NgModule :

```ts
imports: [
  NgOptimizedImage,
  // ...
],
```

</docs-step>
<docs-step title="(Facultatif) Configurer un loader {#optional-set-up-a-loader}">
Un loader d'images n'est pas **obligatoire** pour utiliser NgOptimizedImage, mais l'utiliser avec un CDN d'images permet de bénéficier de puissantes fonctionnalités de performance, notamment des `srcset` automatiques pour vos images.

Un bref guide de configuration d'un loader est disponible dans la section [Configurer un loader d'images](#configuring-an-image-loader-for-ngoptimizedimage) à la fin de cette page.
</docs-step>
<docs-step title="Activer la directive {#enable-the-directive}">
Pour activer la directive `NgOptimizedImage`, remplacez l'attribut `src` de votre image par `ngSrc`.

```html
<img ngSrc="cat.jpg" />
```

Si vous utilisez un [loader tiers intégré](#built-in-loaders), veillez à omettre le chemin de l'URL de base dans `src`, car le loader l'ajoutera automatiquement en préfixe.
</docs-step>
<docs-step title="Marquer les images comme `priority` {#mark-images-as-priority}">
Marquez toujours l'[image LCP](https://web.dev/lcp/#what-elements-are-considered) de votre page comme `priority` pour rendre son chargement prioritaire.

```html
<img ngSrc="cat.jpg" width="400" height="200" priority />
```

Marquer une image comme `priority` applique les optimisations suivantes :

- Définit `fetchpriority=high` (pour en savoir plus sur les priority hints, voir [ici](https://web.dev/priority-hints))
- Définit `loading=eager` (pour en savoir plus sur le lazy loading natif, voir [ici](https://web.dev/browser-level-image-lazy-loading))
- Génère automatiquement un [élément link de preload](https://developer.mozilla.org/docs/Web/HTML/Link_types/preload) en cas de [rendu sur le serveur](guide/ssr).

Angular affiche un avertissement pendant le développement si l'élément LCP est une image dépourvue de l'attribut `priority`. L'élément LCP d'une page peut varier selon plusieurs facteurs, comme les dimensions de l'écran de l'utilisateur ; une page peut donc avoir plusieurs images à marquer comme `priority`. Consultez [CSS for Web Vitals](https://web.dev/css-web-vitals/#images-and-largest-contentful-paint-lcp) pour plus de détails.
</docs-step>
<docs-step title="Indiquer la largeur et la hauteur {#include-width-and-height}">
Afin d'éviter les [décalages de mise en page liés aux images](https://web.dev/css-web-vitals/#images-and-layout-shifts), NgOptimizedImage exige que vous indiquiez une hauteur et une largeur pour votre image, comme suit :

```html
<img ngSrc="cat.jpg" width="400" height="200" />
```

Pour les **images responsives** (images stylées pour s'agrandir et se réduire en fonction du viewport), les attributs `width` et `height` doivent correspondre à la taille intrinsèque du fichier image. Pour les images responsives, il est également important de [définir une valeur pour `sizes`.](#responsive-images)

Pour les **images de taille fixe**, les attributs `width` et `height` doivent refléter la taille de rendu souhaitée de l'image. Le ratio d'aspect de ces attributs doit toujours correspondre au ratio d'aspect intrinsèque de l'image.

NOTE: Si vous ne connaissez pas la taille de vos images, envisagez d'utiliser le « mode fill » pour hériter de la taille du conteneur parent, comme décrit ci-dessous.
</docs-step>
</docs-workflow>

## Utiliser le mode `fill` {#using-fill-mode}

Lorsque vous voulez qu'une image remplisse l'élément qui la contient, vous pouvez utiliser l'attribut `fill`. C'est souvent utile pour obtenir un comportement d'« image d'arrière-plan ». Cela peut aussi être utile lorsque vous ne connaissez pas la largeur et la hauteur exactes de votre image, mais que vous disposez d'un conteneur parent de taille connue dans lequel vous souhaitez faire tenir votre image (voir « object-fit » ci-dessous).

Lorsque vous ajoutez l'attribut `fill` à votre image, vous n'avez pas besoin d'inclure `width` et `height`, et vous ne devez pas le faire, comme dans cet exemple :

```html
<img ngSrc="cat.jpg" fill />
```

Vous pouvez utiliser la propriété CSS [object-fit](https://developer.mozilla.org/docs/Web/CSS/object-fit) pour modifier la manière dont l'image remplit son conteneur. Si vous stylez votre image avec `object-fit: "contain"`, l'image conservera son ratio d'aspect et sera « letterboxée » pour tenir dans l'élément. Si vous définissez `object-fit: "cover"`, l'élément conservera son ratio d'aspect, remplira entièrement l'élément, et une partie du contenu pourra être « rognée ».

Consultez des exemples visuels de ce qui précède dans la [documentation MDN sur object-fit.](https://developer.mozilla.org/docs/Web/CSS/object-fit)

Vous pouvez aussi styler votre image avec la [propriété object-position](https://developer.mozilla.org/docs/Web/CSS/object-position) pour ajuster sa position dans l'élément qui la contient.

IMPORTANT: Pour que l'image « fill » s'affiche correctement, son élément parent **doit** être stylé avec `position: "relative"`, `position: "fixed"` ou `position: "absolute"`.

## Comment migrer votre image d'arrière-plan {#how-to-migrate-your-background-image}

Voici une procédure simple, étape par étape, pour migrer de `background-image` vers `NgOptimizedImage`. Dans ces étapes, nous appellerons « élément conteneur » l'élément qui possède une image d'arrière-plan :

1. Supprimez le style `background-image` de l'élément conteneur.
2. Assurez-vous que l'élément conteneur a `position: "relative"`, `position: "fixed"` ou `position: "absolute"`.
3. Créez un nouvel élément image comme enfant de l'élément conteneur, en utilisant `ngSrc` pour activer la directive `NgOptimizedImage`.
4. Ajoutez l'attribut `fill` à cet élément. N'incluez pas `height` ni `width`.
5. Si vous pensez que cette image pourrait être votre [élément LCP](https://web.dev/lcp/), ajoutez l'attribut `priority` à l'élément image.

Vous pouvez ajuster la manière dont l'image d'arrière-plan remplit le conteneur comme décrit dans la section [Utiliser le mode fill](#using-fill-mode).

## Utiliser des placeholders {#using-placeholders}

### Placeholders automatiques {#automatic-placeholders}

NgOptimizedImage peut afficher un placeholder automatique en basse résolution pour votre image si vous utilisez un CDN ou un hébergeur d'images qui fournit un redimensionnement automatique des images. Tirez parti de cette fonctionnalité en ajoutant l'attribut `placeholder` à votre image :

```html
<img ngSrc="cat.jpg" width="400" height="200" placeholder />
```

L'ajout de cet attribut demande automatiquement une seconde version, plus petite, de l'image à l'aide du loader d'images que vous avez spécifié. Cette petite image est appliquée comme style `background-image` avec un flou CSS pendant le chargement de votre image. Si aucun loader d'images n'est fourni, aucune image de placeholder ne peut être générée et une erreur est levée.

La taille par défaut des placeholders générés est de 30px de large. Vous pouvez modifier cette taille en indiquant une valeur en pixels dans le provider `IMAGE_CONFIG`, comme ci-dessous :

```ts
providers: [
  {
    provide: IMAGE_CONFIG,
    useValue: {
      placeholderResolution: 40
    }
  },
],
```

Si vous voulez des bords nets autour de votre placeholder flouté, vous pouvez envelopper votre image dans une `<div>` conteneur avec le style `overflow: hidden`. Tant que la `<div>` a la même taille que l'image (par exemple en utilisant le style `width: fit-content`), les « bords flous » du placeholder seront masqués.

### Placeholders en data URL {#data-url-placeholders}

Vous pouvez aussi spécifier un placeholder à l'aide d'une [data URL](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URLs) en base64, sans loader d'images. Le format de la data URL est `data:image/[imagetype];[data]`, où `[imagetype]` est le format de l'image, comme `png`, et `[data]` est l'encodage en base64 de l'image. Cet encodage peut être effectué en ligne de commande ou en JavaScript. Pour les commandes précises, consultez [la documentation MDN](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URLs#encoding_data_into_base64_format). Voici un exemple de placeholder en data URL, avec des données tronquées :

```html
<img ngSrc="cat.jpg" width="400" height="200" placeholder="data:image/png;base64,iVBORw0K..." />
```

Cependant, les data URL volumineuses augmentent la taille de vos bundles Angular et ralentissent le chargement de la page. Si vous ne pouvez pas utiliser de loader d'images, l'équipe Angular recommande de garder les images de placeholder en base64 sous 4 Ko et de les utiliser exclusivement pour les images critiques. En plus de réduire les dimensions du placeholder, envisagez de changer le format d'image ou les paramètres utilisés lors de l'enregistrement des images. À très basse résolution, ces paramètres peuvent avoir un effet important sur la taille du fichier.

### Placeholders non floutés {#non-blurred-placeholders}

Par défaut, NgOptimizedImage applique un effet de flou CSS aux placeholders d'image. Pour afficher un placeholder sans flou, fournissez un argument `placeholderConfig` avec un objet qui inclut la propriété `blur` définie à false. Par exemple :

```html
<img ngSrc="cat.jpg" width="400" height="200" placeholder [placeholderConfig]="{blur: false}" />
```

## Ajuster le style des images {#adjusting-image-styling}

Selon le style de l'image, l'ajout des attributs `width` et `height` peut modifier le rendu de l'image. `NgOptimizedImage` vous avertit si le style de votre image l'affiche avec un ratio d'aspect déformé.

Vous pouvez généralement corriger cela en ajoutant `height: auto` ou `width: auto` aux styles de votre image. Pour plus d'informations, consultez l'[article web.dev sur la balise `<img>`](https://web.dev/patterns/web-vitals-patterns/images/img-tag).

Si les attributs `width` et `height` de l'image vous empêchent de la dimensionner comme vous le souhaitez en CSS, envisagez plutôt d'utiliser le mode `fill` et de styler l'élément parent de l'image.

## Fonctionnalités de performance {#performance-features}

NgOptimizedImage inclut un certain nombre de fonctionnalités conçues pour améliorer les performances de chargement de votre application. Ces fonctionnalités sont décrites dans cette section.

### Ajouter des resource hints {#add-resource-hints}

Un [resource hint `preconnect`](https://web.dev/preconnect-and-dns-prefetch) pour l'origine de vos images garantit que l'image LCP se charge le plus rapidement possible.

Les liens de preconnect sont générés automatiquement pour les domaines fournis en argument à un [loader](#optional-set-up-a-loader). Si l'origine d'une image ne peut pas être identifiée automatiquement et qu'aucun lien de preconnect n'est détecté pour l'image LCP, `NgOptimizedImage` émet un avertissement pendant le développement. Dans ce cas, vous devez ajouter manuellement un resource hint à `index.html`. Dans le `<head>` du document, ajoutez une balise `link` avec `rel="preconnect"`, comme ci-dessous :

```html
<link rel="preconnect" href="https://my.cdn.origin" />
```

Pour désactiver les avertissements de preconnect, injectez le jeton `PRECONNECT_CHECK_BLOCKLIST` :

```ts

providers: [
{provide: PRECONNECT_CHECK_BLOCKLIST, useValue: 'https://your-domain.com'}
],

```

Pour plus d'informations sur la génération automatique du preconnect, voir [ici](#why-is-a-preconnect-element-not-being-generated-for-my-image-domain).

### Demander les images à la bonne taille avec le `srcset` automatique {#request-images-at-the-correct-size-with-automatic-srcset}

Définir un [attribut `srcset`](https://developer.mozilla.org/docs/Web/API/HTMLImageElement/srcset) garantit que le navigateur demande une image à la bonne taille pour le viewport de votre utilisateur, afin de ne pas perdre de temps à télécharger une image trop grande. `NgOptimizedImage` génère un `srcset` approprié pour l'image, en fonction de la présence et de la valeur de l'[attribut `sizes`](https://developer.mozilla.org/docs/Web/API/HTMLImageElement/sizes) sur la balise image.

#### Images de taille fixe {#fixed-size-images}

Si votre image doit avoir une taille « fixe » (c'est-à-dire la même taille sur tous les appareils, à la [densité de pixels](https://web.dev/codelab-density-descriptors/) près), il n'est pas nécessaire de définir un attribut `sizes`. Un `srcset` peut être généré automatiquement à partir des attributs width et height de l'image, sans autre information requise.

Exemple de srcset généré :

```html
<img ... srcset="image-400w.jpg 1x, image-800w.jpg 2x" />
```

#### Images responsives {#responsive-images}

Si votre image doit être responsive (c'est-à-dire s'agrandir et se réduire selon la taille du viewport), vous devrez définir un [attribut `sizes`](https://developer.mozilla.org/docs/Web/API/HTMLImageElement/sizes) pour générer le `srcset`.

Si vous n'avez jamais utilisé `sizes`, un bon point de départ consiste à le définir en fonction de la largeur du viewport. Par exemple, si votre CSS fait occuper à l'image 100 % de la largeur du viewport, définissez `sizes` à `100vw` et le navigateur sélectionnera l'image du `srcset` la plus proche de la largeur du viewport (après prise en compte de la densité de pixels). Si votre image n'occupera probablement que la moitié de l'écran (par ex. dans une barre latérale), définissez `sizes` à `50vw` pour que le navigateur sélectionne une image plus petite. Et ainsi de suite.

Si ce qui précède ne couvre pas le comportement souhaité pour votre image, consultez la documentation sur les [valeurs avancées de sizes](#advanced-sizes-values).

Notez que pour les images chargées en lazy loading, `NgOptimizedImage` ajoute automatiquement `"auto"` en préfixe à la valeur `sizes` fournie. Cette optimisation améliore la précision de la sélection dans le srcset sur les navigateurs qui prennent en charge `sizes="auto"`, et elle est ignorée par ceux qui ne la prennent pas en charge. Les images qui ne sont pas chargées en lazy loading, comme les images `priority`, conservent la valeur `sizes` telle que fournie.

Par défaut, les points de rupture responsives sont :

`[16, 32, 48, 64, 96, 128, 256, 384, 640, 750, 828, 1080, 1200, 1920, 2048, 3840]`

Si vous souhaitez personnaliser ces points de rupture, vous pouvez le faire avec le provider `IMAGE_CONFIG` :

```ts
providers: [
  {
    provide: IMAGE_CONFIG,
    useValue: {
      breakpoints: [16, 48, 96, 128, 384, 640, 750, 828, 1080, 1200, 1920]
    }
  },
],
```

Si vous souhaitez définir manuellement un attribut `srcset`, vous pouvez fournir le vôtre avec l'attribut `ngSrcset` :

```html
<img ngSrc="hero.jpg" ngSrcset="100w, 200w, 300w" />
```

Si l'attribut `ngSrcset` est présent, `NgOptimizedImage` génère et définit le `srcset` en fonction des tailles indiquées. N'incluez pas de noms de fichiers d'images dans `ngSrcset` : la directive déduit cette information de `ngSrc`. La directive prend en charge à la fois les descripteurs de largeur (par ex. `100w`) et les descripteurs de densité (par ex. `1x`).

```html
<img ngSrc="hero.jpg" ngSrcset="100w, 200w, 300w" sizes="50vw" />
```

### Désactiver la génération automatique du srcset {#disabling-automatic-srcset-generation}

Pour désactiver la génération du srcset pour une seule image, vous pouvez ajouter l'attribut `disableOptimizedSrcset` sur l'image :

```html
<img ngSrc="about.jpg" disableOptimizedSrcset />
```

### Désactiver le lazy loading des images {#disabling-image-lazy-loading}

Par défaut, `NgOptimizedImage` définit `loading=lazy` pour toutes les images qui ne sont pas marquées `priority`. Vous pouvez désactiver ce comportement pour les images non prioritaires en définissant l'attribut `loading`. Cet attribut accepte les valeurs `eager`, `auto` et `lazy`. [Consultez la documentation de l'attribut standard `loading` des images pour plus de détails](https://developer.mozilla.org/docs/Web/API/HTMLImageElement/loading#value).

```html
<img ngSrc="cat.jpg" width="400" height="200" loading="eager" />
```

### Contrôler le décodage des images {#controlling-image-decoding}

Par défaut, `NgOptimizedImage` définit `decoding="auto"` pour toutes les images. Cela permet au navigateur de choisir le moment optimal pour décoder une image après l'avoir récupérée. Lorsqu'une image est marquée comme `priority`, Angular définit automatiquement `decoding="sync"` pour que l'image soit décodée et affichée le plus tôt possible, ce qui aide à améliorer les performances du **Largest Contentful Paint (LCP)**.

Vous pouvez remplacer la valeur par défaut `auto` pour les images non prioritaires en définissant explicitement l'attribut `decoding`. Les images prioritaires utilisent toujours `decoding="sync"`.  
[Consultez la documentation de l'attribut standard `decoding` des images pour plus de détails](https://developer.mozilla.org/docs/Web/HTML/Element/img#decoding).

```html
<!-- Default: decoding is 'auto' -->
<img ngSrc="gallery/landscape.jpg" width="1200" height="800" />

<!-- Decode the image asynchronously to avoid blocking the main thread.-->
<img ngSrc="gallery/preview.jpg" width="600" height="400" decoding="async" />

<!-- Priority images automatically use decoding="sync" -->
<img ngSrc="awesome.jpg" width="500" height="625" priority />

<!-- Decode immediately (can block) when you need the pixels right away -->
<img ngSrc="hero.jpg" width="1600" height="900" decoding="sync" />
```

**Valeurs autorisées**

- `auto` (par défaut) : laisse le navigateur choisir la stratégie optimale.
- `async` : décode l'image de manière asynchrone, en évitant si possible de bloquer le thread principal.
- `sync` : décode l'image immédiatement ; peut bloquer le rendu, mais garantit que les pixels sont prêts dès que l'image est disponible.

### Valeurs avancées de 'sizes' {#advanced-sizes-values}

Vous pouvez vouloir afficher des images avec des largeurs variables selon la taille des écrans. Un exemple courant de ce pattern est une mise en page en grille ou en colonnes qui affiche une seule colonne sur les appareils mobiles et deux colonnes sur les appareils plus grands. Vous pouvez exprimer ce comportement dans l'attribut `sizes`, à l'aide d'une syntaxe de « media query », comme ci-dessous :

```html
<img ngSrc="cat.jpg" width="400" height="200" sizes="(max-width: 768px) 100vw, 50vw" />
```

Dans l'exemple ci-dessus, l'attribut `sizes` signifie : « Je m'attends à ce que cette image occupe 100 % de la largeur de l'écran sur les appareils de moins de 768px de large. Sinon, je m'attends à ce qu'elle occupe 50 % de la largeur de l'écran. »

Pour plus d'informations sur l'attribut `sizes`, consultez [web.dev](https://web.dev/learn/design/responsive-images/#sizes) ou [mdn](https://developer.mozilla.org/docs/Web/API/HTMLImageElement/sizes).

## Configurer un loader d'images pour `NgOptimizedImage` {#configuring-an-image-loader-for-ngoptimizedimage}

Un « loader » est une fonction qui génère une [URL de transformation d'image](https://web.dev/image-cdns/#how-image-cdns-use-urls-to-indicate-optimization-options) pour un fichier image donné. Lorsque c'est approprié, `NgOptimizedImage` définit les transformations de taille, de format et de qualité d'une image.

`NgOptimizedImage` fournit à la fois un loader générique qui n'applique aucune transformation, et des loaders pour divers services d'images tiers. La directive permet aussi d'écrire votre propre loader personnalisé.

| Type de loader                            | Comportement                                                                                                                                                                                                                       |
| :------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Loader générique                         | L'URL renvoyée par le loader générique correspond toujours à la valeur de `src`. Autrement dit, ce loader n'applique aucune transformation. Les sites qui utilisent Angular pour servir des images constituent le principal cas d'usage prévu pour ce loader. |
| Loaders pour les services d'images tiers | L'URL renvoyée par les loaders des services d'images tiers suit les conventions d'API propres à ce service d'images.                                                                                              |
| Loaders personnalisés                         | Le comportement d'un loader personnalisé est défini par son développeur. Utilisez un loader personnalisé si votre service d'images n'est pas pris en charge par les loaders préconfigurés fournis avec `NgOptimizedImage`.                                   |

En fonction des services d'images couramment utilisés avec les applications Angular, `NgOptimizedImage` fournit des loaders préconfigurés pour fonctionner avec les services d'images suivants :

| Service d'images             | API Angular               | Documentation                                                               |
| :------------------------ | :------------------------ | :-------------------------------------------------------------------------- |
| Cloudflare Image Resizing | `provideCloudflareLoader` | [Documentation](https://developers.cloudflare.com/images/image-resizing/)   |
| Cloudinary                | `provideCloudinaryLoader` | [Documentation](https://cloudinary.com/documentation/resizing_and_cropping) |
| ImageKit                  | `provideImageKitLoader`   | [Documentation](https://docs.imagekit.io/)                                  |
| Imgix                     | `provideImgixLoader`      | [Documentation](https://docs.imgix.com/)                                    |
| Netlify                   | `provideNetlifyLoader`    | [Documentation](https://docs.netlify.com/image-cdn/overview/)               |

Pour utiliser le **loader générique**, aucune modification de code supplémentaire n'est nécessaire. C'est le comportement par défaut.

### Loaders intégrés {#built-in-loaders}

Pour utiliser un loader existant pour un **service d'images tiers**, ajoutez la fabrique de provider du service choisi au tableau `providers`. Dans l'exemple ci-dessous, le loader Imgix est utilisé :

```ts
providers: [
  provideImgixLoader('https://my.base.url/'),
],
```

L'URL de base de vos ressources d'images doit être passée en argument à la fabrique de provider. Pour la plupart des sites, cette URL de base doit correspondre à l'un des motifs suivants :

- <https://yoursite.yourcdn.com>
- <https://subdomain.yoursite.com>
- <https://subdomain.yourcdn.com/yoursite>

Pour en savoir plus sur la structure de l'URL de base, consultez la documentation du fournisseur de CDN correspondant.

### Loaders personnalisés {#custom-loaders}

Pour utiliser un **loader personnalisé**, fournissez votre fonction de loader comme valeur du jeton d'injection `IMAGE_LOADER`. Dans l'exemple ci-dessous, la fonction de loader personnalisée renvoie une URL commençant par `https://example.com` qui inclut `src`, `width` et `height` comme paramètres d'URL.

```ts
providers: [
  {
    provide: IMAGE_LOADER,
    useValue: (config: ImageLoaderConfig) => {
      return `https://example.com/images?src=${config.src}&width=${config.width}&height=${config.height}`;
    },
  },
],
```

Une fonction de loader pour la directive `NgOptimizedImage` prend en argument un objet de type `ImageLoaderConfig` (issu de `@angular/common`) et renvoie l'URL absolue de la ressource image. L'objet `ImageLoaderConfig` contient la propriété `src`, ainsi que les propriétés facultatives `width`, `height` et `loaderParams`.

NOTE: même si la propriété `width` n'est pas toujours présente, un loader personnalisé doit l'utiliser pour permettre de demander des images de différentes largeurs, afin que `ngSrcset` fonctionne correctement.

### La propriété `loaderParams` {#the-loaderparams-property}

La directive `NgOptimizedImage` prend en charge un attribut supplémentaire, appelé `loaderParams`, spécialement conçu pour l'utilisation de loaders personnalisés. L'attribut `loaderParams` prend comme valeur un objet avec des propriétés quelconques, et ne fait rien par lui-même. Les données de `loaderParams` sont ajoutées à l'objet `ImageLoaderConfig` passé à votre loader personnalisé, et peuvent servir à contrôler le comportement du loader.

Un usage courant de `loaderParams` consiste à contrôler les fonctionnalités avancées d'un CDN d'images.

### Utiliser la propriété `transform` avec les loaders intégrés {#using-the-transform-property-with-built-in-loaders}

Les loaders intégrés pour Cloudinary, Cloudflare, ImageKit et Imgix prennent en charge une propriété spéciale `transform` dans `loaderParams`. Cette propriété vous permet d'appliquer des transformations d'image personnalisées fournies par votre CDN.

La propriété `transform` accepte deux formats :

#### Format chaîne {#string-format}

Fournissez les transformations sous forme de chaîne séparée par des virgules, en utilisant la syntaxe de transformation de votre CDN :

```html
<img
  ngSrc="my-image.jpg"
  width="400"
  height="300"
  [loaderParams]="{transform: 'e_grayscale,r_10'}"
/>
```

#### Format objet {#object-format}

Fournissez les transformations sous forme d'objet avec des paires clé-valeur.

```html
<img
  ngSrc="my-image.jpg"
  width="400"
  height="300"
  [loaderParams]="{transform: {e: 'grayscale', r: 10}}"
/>
```

NOTE: La propriété `transform` n'est pas prise en charge par le loader Netlify, car le CDN d'images de Netlify ne fournit pas de paramètres de transformation personnalisés.

### Exemple de loader personnalisé {#example-custom-loader}

Voici un exemple de fonction de loader personnalisée. Cette fonction d'exemple concatène `src`, `width` et `height`, et utilise `loaderParams` pour contrôler une fonctionnalité de CDN personnalisée pour les coins arrondis :

```ts
const myCustomLoader = (config: ImageLoaderConfig) => {
  let url = `https://example.com/images/${config.src}?`;
  let queryParams = [];
  if (config.width) {
    queryParams.push(`w=${config.width}`);
  }
  if (config.height) {
    queryParams.push(`h=${config.height}`);
  }
  if (config.loaderParams?.roundedCorners) {
    queryParams.push('mask=corners&corner-radius=5');
  }
  return url + queryParams.join('&');
};
```

Notez que dans l'exemple ci-dessus, nous avons inventé le nom de propriété 'roundedCorners' pour contrôler une fonctionnalité de notre loader personnalisé. Nous pourrions ensuite utiliser cette fonctionnalité lors de la création d'une image, comme suit :

```html
<img ngSrc="profile.jpg" width="300" height="300" [loaderParams]="{roundedCorners: true}" />
```

## Questions fréquentes {#frequently-asked-questions}

### NgOptimizedImage prend-il en charge la propriété CSS `background-image` ? {#does-ngoptimizedimage-support-the-background-image-css-property}

NgOptimizedImage ne prend pas directement en charge la propriété CSS `background-image`, mais la directive est conçue pour s'adapter facilement au cas d'usage d'une image servant d'arrière-plan à un autre élément.

Pour une procédure étape par étape de migration de `background-image` vers `NgOptimizedImage`, consultez la section [Comment migrer votre image d'arrière-plan](#how-to-migrate-your-background-image) ci-dessus.

### Pourquoi ne puis-je pas utiliser `src` avec `NgOptimizedImage` ? {#why-cant-i-use-src-with-ngoptimizedimage}

L'attribut `ngSrc` a été choisi comme déclencheur de NgOptimizedImage pour des raisons techniques liées à la manière dont le navigateur charge les images. NgOptimizedImage modifie programmatiquement l'attribut `loading` : si le navigateur voit l'attribut `src` avant que ces modifications ne soient effectuées, il commence immédiatement à télécharger le fichier image, et les modifications de chargement sont ignorées.

### Pourquoi aucun élément preconnect n'est-il généré pour le domaine de mes images ? {#why-is-a-preconnect-element-not-being-generated-for-my-image-domain}

La génération du preconnect repose sur une analyse statique de votre application. Cela signifie que le domaine des images doit être inclus directement dans le paramètre du loader, comme dans l'exemple suivant :

```ts
providers: [
  provideImgixLoader('https://my.base.url/'),
],
```

Si vous utilisez une variable pour passer la chaîne du domaine au loader, ou si vous n'utilisez pas de loader, l'analyse statique ne pourra pas identifier le domaine et aucun lien de preconnect ne sera généré. Dans ce cas, vous devez ajouter manuellement un lien de preconnect dans le head du document, comme [décrit ci-dessus](#add-resource-hints).

### Puis-je utiliser deux domaines d'images différents dans la même page ? {#can-i-use-two-different-image-domains-in-the-same-page}

Le pattern de provider des [loaders d'images](#configuring-an-image-loader-for-ngoptimizedimage) est conçu pour être aussi simple que possible dans le cas d'usage courant d'un seul CDN d'images utilisé dans un composant. Il reste toutefois tout à fait possible de gérer plusieurs CDN d'images avec un seul provider.

Pour cela, nous recommandons d'écrire un [loader d'images personnalisé](#custom-loaders) qui utilise la [propriété `loaderParams`](#the-loaderparams-property) pour transmettre un indicateur précisant quel CDN d'images utiliser, puis qui invoque le loader approprié en fonction de cet indicateur.

### Pouvez-vous ajouter un nouveau loader intégré pour mon CDN préféré ? {#can-you-add-a-new-built-in-loader-for-my-preferred-cdn}

Pour des raisons de maintenance, nous ne prévoyons pas actuellement de prendre en charge d'autres loaders intégrés dans le dépôt Angular. Nous encourageons plutôt les développeurs à publier tout loader d'images supplémentaire sous forme de packages tiers.

### Puis-je l'utiliser avec la balise `<picture>` ? {#can-i-use-this-with-the-picture-tag}

Non, mais c'est prévu dans notre feuille de route, alors restez à l'écoute.

Si vous attendez cette fonctionnalité, votez pour l'issue GitHub [ici](https://github.com/angular/angular/issues/56594).

### Comment trouver mon image LCP avec les Chrome DevTools ? {#how-do-i-find-my-lcp-image-with-chrome-devtools}

1. Dans l'onglet Performance des Chrome DevTools, cliquez sur le bouton « start profiling and reload page » en haut à gauche. Il ressemble à une icône d'actualisation de page.

2. Cela déclenche un snapshot de profilage de votre application Angular.

3. Une fois le résultat du profilage disponible, sélectionnez « LCP » dans la section des timings.

4. Une entrée de résumé devrait apparaître dans le panneau du bas. Vous trouverez l'élément LCP sur la ligne « related node ». Cliquer dessus affiche l'élément dans le panneau Elements.

<img alt="LCP dans les Chrome DevTools" src="assets/images/guide/image-optimization/devtools-lcp.png">

NOTE: Cela n'identifie que l'élément LCP dans le viewport de la page que vous testez. Il est également recommandé d'utiliser l'émulation mobile pour identifier l'élément LCP sur les écrans plus petits.
