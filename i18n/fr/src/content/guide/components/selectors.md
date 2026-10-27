<!-- source: a04926ba085b9b059b65752dde44fa6c1ae08cc1 -->
# Selectors de composants

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Chaque composant définit
un [selector CSS](https://developer.mozilla.org/docs/Web/CSS/CSS_selectors) qui détermine comment
le composant est utilisé :

```angular-ts {highlight: [2]}
@Component({
  selector: 'profile-photo',
  ...
})
export class ProfilePhoto { }
```

Vous utilisez un composant en créant un élément HTML correspondant dans les templates d'_autres_ composants :

```angular-ts {highlight: [3]}
@Component({
  template: `
    <profile-photo />
    <button>Upload a new profile photo</button>`,
  ...,
})
export class UserProfile { }
```

**Angular fait correspondre les selectors de manière statique, à la compilation**. Modifier le DOM à l'exécution, que ce soit via
des bindings Angular ou avec les API du DOM, n'a aucun effet sur les composants rendus.

**Un élément ne peut correspondre qu'à un seul selector de composant.** Si plusieurs selectors de composants correspondent à un
même élément, Angular signale une erreur.

**Les selectors de composants sont sensibles à la casse.**

## Types de selectors {#types-of-selectors}

Angular prend en charge un sous-ensemble limité
des [types de selectors CSS de base](https://developer.mozilla.org/docs/Web/CSS/CSS_Selectors) dans
les selectors de composants :

| **Type de selector** | **Description** | **Exemples** |
| ------------------ | --------------------------------------------------------------------------------------------------------------- | ----------------------------- |
| Selector de type | Sélectionne les éléments selon leur nom de balise HTML, ou nom de nœud. | `profile-photo` |
| Selector d'attribute | Sélectionne les éléments selon la présence d'un attribut HTML et, de façon facultative, une valeur exacte pour cet attribut. | `[dropzone]` `[type="reset"]` |
| Selector de classe | Sélectionne les éléments selon la présence d'une classe CSS. | `.menu-item` |

Pour les valeurs d'attribut, Angular permet de faire correspondre une valeur d'attribut exacte avec l'opérateur d'égalité (`=`).
Angular ne prend pas en charge les autres opérateurs de valeur d'attribut.

Les selectors de composants Angular ne prennent pas en charge les combinateurs, notamment
le [combinateur de descendance](https://developer.mozilla.org/docs/Web/CSS/Descendant_combinator)
ou le [combinateur d'enfant](https://developer.mozilla.org/docs/Web/CSS/Child_combinator).

Les selectors de composants Angular ne permettent pas
de spécifier des [namespaces](https://developer.mozilla.org/docs/Web/SVG/Namespaces_Crash_Course).

### La pseudo-classe `:not` {#the-not-pseudo-class}

Angular prend en charge [la pseudo-classe `:not`](https://developer.mozilla.org/docs/Web/CSS/:not).
Vous pouvez ajouter cette pseudo-classe à n'importe quel autre selector pour restreindre les éléments auxquels le
selector d'un composant correspond. Par exemple, vous pourriez définir un selector d'attribute `[dropzone]` et empêcher
la correspondance avec les éléments `textarea` :

```angular-ts {highlight: [2]}
@Component({
  selector: '[dropzone]:not(textarea)',
  ...
})
export class DropZone { }
```

Angular ne prend en charge aucune autre pseudo-classe ni aucun pseudo-élément dans les selectors de composants.

### Combiner des selectors {#combining-selectors}

Vous pouvez combiner plusieurs selectors en les concaténant. Par exemple, vous pouvez cibler les éléments `<button>`
qui spécifient `type="reset"` :

```angular-ts {highlight: [2]}
@Component({
  selector: 'button[type="reset"]',
  ...
})
export class ResetButton { }
```

Vous pouvez aussi définir plusieurs selectors avec une liste séparée par des virgules :

```angular-ts {highlight: [2]}
@Component({
  selector: 'drop-zone, [dropzone]',
  ...
})
export class DropZone { }
```

Angular crée un composant pour chaque élément qui correspond à _l'un quelconque_ des selectors de la liste.

## Choisir un selector {#choosing-a-selector}

La grande majorité des composants devraient utiliser un nom de custom element comme selector. Tous les noms
de custom elements devraient inclure un tiret, comme le décrit
[la spécification HTML](https://html.spec.whatwg.org/multipage/custom-elements.html#valid-custom-element-name).
Par défaut, Angular signale une erreur s'il rencontre un nom de balise personnalisé qui ne correspond à aucun
composant disponible, ce qui évite les bugs dus à des noms de composants mal orthographiés.

Consultez [Configuration avancée des composants](guide/components/advanced-configuration) pour plus de détails sur
l'utilisation des [custom elements natifs](https://developer.mozilla.org/docs/Web/Web_Components) dans
les templates Angular.

### Préfixes de selectors {#selector-prefixes}

L'équipe Angular recommande d'utiliser un préfixe court et cohérent pour tous les composants personnalisés
définis dans votre projet. Par exemple, si vous deviez construire YouTube avec Angular, vous pourriez
préfixer vos composants avec `yt-`, avec des composants comme `yt-menu`, `yt-player`, etc. Placer
vos selectors dans un namespace de cette façon indique immédiatement d'où vient un composant donné. Par
défaut, l'Angular CLI utilise `app-`.

IMPORTANT: Angular utilise le préfixe de selector `ng` pour ses propres API du framework. N'utilisez jamais `ng` comme préfixe de selector pour vos propres composants personnalisés.

### Quand utiliser un selector d'attribute {#when-to-use-an-attribute-selector}

Envisagez un selector d'attribute quand vous souhaitez créer un composant sur un élément natif
standard. Par exemple, si vous souhaitez créer un composant de bouton personnalisé, vous pouvez tirer parti de
l'élément standard `<button>` en utilisant un selector d'attribute :

```angular-ts {highlight: [2]}
@Component({
  selector: 'button[yt-upload]',
   ...
})
export class YouTubeUploadButton { }
```

Cette approche permet aux consommateurs du composant d'utiliser directement toutes les API standard de l'élément
sans travail supplémentaire. C'est particulièrement utile pour les attributs ARIA comme `aria-label`.

Angular ne signale pas d'erreur quand il rencontre des attributs personnalisés qui ne correspondent à aucun
composant disponible. Lorsqu'ils utilisent des composants avec des selectors d'attribute, les consommateurs peuvent oublier d'importer le
composant ou son NgModule, ce qui empêche le rendu du composant.
Consultez [Importer et utiliser des composants](guide/components#imports-in-the-component-decorator) pour plus d'informations.

Les composants qui définissent des selectors d'attribute devraient utiliser des attributs en minuscules et en dash-case. Vous pouvez
suivre la même recommandation de préfixe décrite ci-dessus.
