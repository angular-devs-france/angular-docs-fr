<!-- source: 52178220a27826a738b5e97c0130bd7e2d1d9710 -->
# Les composants dans Angular

Les composants sont les briques fondamentales de toute application Angular. Chaque composant comporte trois parties :

- Une classe TypeScript
- Un template HTML
- Des styles CSS

NOTE: Pour en savoir plus, consultez [les composants dans le guide des Fondamentaux](/essentials/components).

Dans cette activité, vous allez apprendre à mettre à jour le template et les styles d'un composant.

<hr />

C'est une excellente occasion de faire vos premiers pas avec Angular.

<docs-workflow>

<docs-step title="Mettre à jour le template du composant {#update-the-component-template}">
Mettez à jour la propriété `template` pour qu'elle affiche `Hello Universe`

```ts
template: `
  Hello Universe
`,
```

Quand vous avez modifié le template HTML, l'aperçu s'est mis à jour avec votre message. Allons un peu plus loin : changez la couleur du texte.
</docs-step>

<docs-step title="Mettre à jour les styles du composant {#update-the-component-styles}">
Mettez à jour la valeur des styles et changez la propriété `color` de `blue` à `#a144eb`.

```ts
styles: `
  :host {
    color: #a144eb;
  }
`,
```

En regardant l'aperçu, vous verrez que la couleur du texte a changé.
</docs-step>

</docs-workflow>

Dans Angular, vous pouvez utiliser tout le CSS et le HTML pris en charge par le navigateur. Si vous le souhaitez, vous pouvez placer votre template et vos styles dans des fichiers séparés.
