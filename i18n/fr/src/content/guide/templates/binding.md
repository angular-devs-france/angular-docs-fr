<!-- source: 08d576cd3e3c3a73478fdfbb00cfe18168555975 -->
# Lier du texte, des propriétés et des attributs dynamiques

Dans Angular, un **binding** crée une connexion dynamique entre le template d'un composant et ses données. Cette connexion garantit que les modifications des données du composant mettent automatiquement à jour le template rendu.

## Afficher du texte dynamique avec l'interpolation de texte {#render-dynamic-text-with-text-interpolation}

Vous pouvez lier du texte dynamique dans les templates avec des doubles accolades, ce qui indique à Angular qu'il est responsable de l'expression qu'elles contiennent et de sa bonne mise à jour. C'est ce qu'on appelle l'**interpolation de texte**.

```angular-ts
@Component({
  template: `
    <p>Your color preference is {{ theme }}.</p>
  `,
  ...
})
export class App {
  theme = 'dark';
}
```

Dans cet exemple, quand l'extrait est rendu dans la page, Angular remplace `{{ theme }}` par `dark`.

```angular-html
<!-- Rendered Output -->
<p>Your color preference is dark.</p>
```

Les bindings qui changent au fil du temps doivent lire leurs valeurs depuis des [signals](/guide/signals). Angular suit les signals lus dans le template et met à jour la page rendue quand les valeurs de ces signals changent.

```angular-ts
@Component({
  template: `
    <!-- Does not necessarily update when `welcomeMessage` changes. -->
    <p>{{ welcomeMessage }}</p>

    <p>Your color preference is {{ theme() }}.</p> <!-- Always updates when the value of the `theme` signal changes. -->
  `
  ...
})
export class App {
  welcomeMessage = "Welcome, enjoy this app that we built for you";
  theme = signal('dark');
}
```

Pour plus de détails, consultez le [guide des signals](/guide/signals).

Pour poursuivre l'exemple du thème, si un utilisateur clique sur un bouton qui met à jour le signal `theme` à `'light'` après le chargement de la page, la page se met à jour en conséquence :

```angular-html
<!-- Rendered Output -->
<p>Your color preference is light.</p>
```

Vous pouvez utiliser l'interpolation de texte partout où vous écririez normalement du texte en HTML.

Toutes les valeurs d'expression sont converties en chaîne de caractères. Les objets et les tableaux sont convertis à l'aide de la méthode `toString` de la valeur.

## Lier des propriétés et des attributs dynamiques {#binding-dynamic-properties-and-attributes}

Angular permet de lier des valeurs dynamiques à des propriétés d'objets et à des attributs HTML avec des crochets.

Vous pouvez vous lier aux propriétés de l'instance DOM d'un élément HTML, d'une instance de [composant](/guide/components) ou d'une instance de [directive](/guide/directives).

### Propriétés des éléments natifs {#native-element-properties}

Chaque élément HTML possède une représentation DOM correspondante. Par exemple, chaque élément HTML `<button>` correspond à une instance de `HTMLButtonElement` dans le DOM. Dans Angular, vous utilisez les property bindings pour affecter des valeurs directement à la représentation DOM de l'élément.

```angular-html
<!-- Bind the `disabled` property on the button element's DOM object -->
<button [disabled]="isFormValid()">Save</button>
```

Dans cet exemple, chaque fois que `isFormValid` change, Angular définit automatiquement la propriété `disabled` de l'instance de `HTMLButtonElement`.

### Propriétés de composants et de directives {#component-and-directive-properties}

Quand un élément est un composant Angular, vous pouvez utiliser les property bindings pour définir les propriétés d'input du composant avec la même syntaxe à crochets.

```angular-html
<!-- Bind the `value` property on the `MyListbox` component instance. -->
<my-listbox [value]="mySelection()" />
```

Dans cet exemple, chaque fois que `mySelection` change, Angular définit automatiquement la propriété `value` de l'instance de `MyListbox`.

Vous pouvez également vous lier aux propriétés des directives.

```angular-html
<!-- Bind to the `ngSrc` property of the `NgOptimizedImage` directive  -->
<img [ngSrc]="profilePhotoUrl()" alt="The current user's profile photo" />
```

### Attributs {#attributes}

Quand vous devez définir des attributs HTML qui n'ont pas de propriété DOM correspondante, comme les attributs SVG, vous pouvez lier des attributs aux éléments de votre template avec le préfixe `attr.`.

<!-- prettier-ignore -->
```angular-html
<!-- Bind the `role` attribute on the `<ul>` element to the component's `listRole` property. -->
<ul [attr.role]="listRole()">
```

Dans cet exemple, chaque fois que `listRole` change, Angular définit automatiquement l'attribut `role` de l'élément `<ul>` en appelant `setAttribute`.

Si la valeur d'un attribute binding est `null`, Angular supprime l'attribut en appelant `removeAttribute`.

### Interpolation de texte dans les propriétés et les attributs {#text-interpolation-in-properties-and-attributes}

Vous pouvez aussi utiliser la syntaxe d'interpolation de texte dans les propriétés et les attributs, en utilisant la syntaxe à doubles accolades au lieu de placer des crochets autour du nom de la propriété ou de l'attribut. Avec cette syntaxe, Angular traite l'affectation comme un property binding.

```angular-html
<!-- Binds a value to the `alt` property of the image element's DOM object. -->
<img src="profile-photo.jpg" alt="Profile photo of {{ firstName() }}" />
```

## Bindings de classes CSS et de propriétés de style {#css-class-and-style-property-bindings}

Angular offre des fonctionnalités supplémentaires pour lier des classes CSS et des propriétés de style CSS aux éléments.

### Classes CSS {#css-classes}

Vous pouvez créer un class binding CSS pour ajouter ou retirer une classe CSS d'un élément selon que la valeur liée est [truthy ou falsy](https://developer.mozilla.org/en-US/docs/Glossary/Truthy).

<!-- prettier-ignore -->
```angular-html
<!-- When `isExpanded` is truthy, add the `expanded` CSS class. -->
<ul [class.expanded]="isExpanded()">
```

Vous pouvez aussi vous lier directement à la propriété `class`. Angular accepte trois types de valeur :

| Description de la valeur de `class` | Type TypeScript |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| Une chaîne contenant une ou plusieurs classes CSS séparées par des espaces | `string` |
| Un tableau de chaînes de classes CSS | `string[]` |
| Un objet dont chaque nom de propriété est un nom de classe CSS et dont chaque valeur correspondante détermine, selon qu'elle est truthy ou non, si cette classe est appliquée à l'élément. | `Record<string, any>` |

```angular-ts
@Component({
  template: `
    <ul [class]="listClasses"> ... </ul>
    <section [class]="sectionClasses()"> ... </section>
    <button [class]="buttonClasses()"> ... </button>
  `,
  ...
})
export class UserProfile {
  listClasses = 'full-width outlined';
  sectionClasses = signal(['expandable', 'elevated']);
  buttonClasses = signal({
    highlighted: true,
    embiggened: false,
  });
}
```

L'exemple ci-dessus produit le DOM suivant :

<!-- prettier-ignore -->
```angular-html
<ul class="full-width outlined"> ... </ul>
<section class="expandable elevated"> ... </section>
<button class="highlighted"> ... </button>
```

Angular ignore toute valeur de chaîne qui n'est pas un nom de classe CSS valide.

Quand vous utilisez à la fois des classes CSS statiques, un binding direct à `class` et des bindings à des classes spécifiques, Angular combine intelligemment toutes les classes dans le résultat rendu.

```angular-ts
@Component({
  template: `<ul class="list" [class]="listType()" [class.expanded]="isExpanded()"> ...`,
  ...
})
export class Listbox {
  listType = signal('box');
  isExpanded = signal(true);
}
```

Dans l'exemple ci-dessus, Angular rend l'élément `ul` avec les trois classes CSS.

<!-- prettier-ignore -->
```angular-html
<ul class="list box expanded">
```

Angular ne garantit aucun ordre particulier des classes CSS sur les éléments rendus.

Quand vous liez `class` à un tableau ou à un objet, Angular compare la valeur précédente à la valeur actuelle avec l'opérateur d'égalité stricte (`===`). Vous devez créer une nouvelle instance d'objet ou de tableau quand vous modifiez ces valeurs pour qu'Angular applique les mises à jour.

Si un élément possède plusieurs bindings pour la même classe CSS, Angular résout les conflits en suivant son ordre de priorité des styles.

NOTE: Les class bindings ne prennent pas en charge plusieurs noms de classe séparés par des espaces dans une même clé. Ils ne prennent pas non plus en charge les mutations d'objets, car la référence du binding reste la même. Si vous avez besoin de l'un ou de l'autre, utilisez la directive [ngClass](/api/common/NgClass).

### Propriétés de style CSS {#css-style-properties}

Vous pouvez aussi vous lier directement aux propriétés de style CSS d'un élément.

<!-- prettier-ignore -->
```angular-html
<!-- Set the CSS `display` property based on the `isExpanded` property. -->
<section [style.display]="isExpanded() ? 'block' : 'none'">
```

Vous pouvez en outre préciser l'unité des propriétés CSS qui en acceptent une.

<!-- prettier-ignore -->
```angular-html
<!-- Set the CSS `height` property to a pixel value based on the `sectionHeightInPixels` property. -->
<section [style.height.px]="sectionHeightInPixels()">
```

Vous pouvez aussi définir plusieurs valeurs de style dans un seul binding. Angular accepte les types de valeur suivants :

| Description de la valeur de `style` | Type TypeScript |
| ------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| Une chaîne contenant zéro, une ou plusieurs déclarations CSS, comme `"display: flex; margin: 8px"`. | `string` |
| Un objet dont chaque nom de propriété est un nom de propriété CSS et dont chaque valeur correspondante est la valeur de cette propriété CSS. | `Record<string, any>` |

```angular-ts
@Component({
  template: `
    <ul [style]="listStyles()"> ... </ul>
    <section [style]="sectionStyles()"> ... </section>
  `,
  ...
})
export class UserProfile {
  listStyles = signal('display: flex; padding: 8px');
  sectionStyles = signal({
    border: '1px solid black',
    'font-weight': 'bold',
  });
}
```

L'exemple ci-dessus produit le DOM suivant.

<!-- prettier-ignore -->
```angular-html
<ul style="display: flex; padding: 8px"> ... </ul>
<section style="border: 1px solid black; font-weight: bold"> ... </section>
```

Quand vous liez `style` à un objet, Angular compare la valeur précédente à la valeur actuelle avec l'opérateur d'égalité stricte (`===`). Vous devez créer une nouvelle instance d'objet quand vous modifiez ces valeurs pour qu'Angular applique les mises à jour.

Si un élément possède plusieurs bindings pour la même propriété de style, Angular résout les conflits en suivant son ordre de priorité des styles.

## Attributs ARIA {#aria-attributes}

Angular permet de lier des valeurs de chaîne aux attributs ARIA.

```angular-html
<button type="button" [aria-label]="actionLabel()">
  {{ actionLabel() }}
</button>
```

Angular écrit la valeur de chaîne dans l'attribut `aria-label` de l'élément et le supprime quand la valeur liée est `null`.

Certaines fonctionnalités ARIA exposent des propriétés DOM ou des inputs de directive qui acceptent des valeurs structurées (comme des références d'éléments). Utilisez les property bindings standard dans ces cas-là. Consultez le [guide d'accessibilité](/best-practices/a11y#aria-attributes-and-properties) pour des exemples et des conseils supplémentaires.
