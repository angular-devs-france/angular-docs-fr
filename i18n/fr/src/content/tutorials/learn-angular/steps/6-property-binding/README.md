<!-- source: 8d47c7874ee82fa8d29e9b8d3895f1e652e2c001 -->
# Le property binding dans Angular

Le property binding dans Angular vous permet de définir les valeurs des propriétés des éléments HTML, des composants Angular et plus encore.

Utilisez le property binding pour définir dynamiquement les valeurs de propriétés et d'attributs. Vous pouvez par exemple activer ou désactiver des fonctionnalités d'un bouton, définir des chemins d'image par programmation et partager des valeurs entre composants.

NOTE: Pour en savoir plus, consultez [la définition de propriétés et d'attributs dynamiques dans le guide des Fondamentaux](/essentials/templates#setting-dynamic-properties-and-attributes).

Dans cette activité, vous allez apprendre à utiliser le property binding dans les templates.

<hr />

Pour lier un attribut d'un élément, entourez le nom de l'attribut de crochets. Voici un exemple :

```angular-html
<img alt="photo" [src]="imageURL" />
```

Dans cet exemple, la valeur de l'attribut `src` sera liée à la propriété de classe `imageURL`. Quelle que soit la valeur de `imageURL`, elle sera définie comme attribut `src` de la balise `img`.

<docs-workflow>

<docs-step title="Ajouter une propriété nommée `isEditable` {#add-a-property-called-iseditable}" header="app.ts" language="ts">
Mettez à jour le code de `app.ts` en ajoutant à la classe `App` une propriété nommée `isEditable`, avec la valeur initiale `true`.

```ts {highlight:[2]}
export class App {
  isEditable = true;
}
```

</docs-step>

<docs-step title="Lier `contentEditable` {#bind-to-contenteditable}" header="app.ts" language="ts">
Ensuite, liez l'attribut `contentEditable` du `div` à la propriété `isEditable` en utilisant la syntaxe <code aria-label="crochets">[]</code>.

```angular-ts {highlight:[3]}
@Component({
  ...
  template: `<div [contentEditable]="isEditable"></div>`,
})
```

</docs-step>

</docs-workflow>

Le div est maintenant modifiable. Beau travail 👍

Le property binding est l'une des nombreuses fonctionnalités puissantes d'Angular. Pour en savoir plus, consultez [la documentation Angular](guide/templates/binding#css-class-and-style-property-bindings).
