<!-- source: e3692f3a04e17aed0c1aceae203dc5b483cb1a7b -->
# Mettre à jour la classe du composant

Dans Angular, la logique et le comportement d'un composant sont définis dans sa classe TypeScript.

NOTE: Pour en savoir plus, consultez [l'affichage de texte dynamique dans le guide des Fondamentaux](/essentials/templates#showing-dynamic-text).

Dans cette activité, vous allez apprendre à mettre à jour la classe du composant et à utiliser [l'interpolation](/guide/templates/binding#render-dynamic-text-with-text-interpolation).

<hr />

<docs-workflow>

<docs-step title="Ajouter une propriété nommée `city` {#add-a-property-called-city}">
Mettez à jour la classe du composant en ajoutant une propriété nommée `city` à la classe `App`.

```ts
export class App {
  city = 'San Francisco';
}
```

La propriété `city` est de type `string`, mais vous pouvez omettre le type grâce à [l'inférence de type de TypeScript](https://www.typescriptlang.org/docs/handbook/type-inference.html). La propriété `city` peut être utilisée dans la classe `App` et référencée dans le template du composant.

<br>

Pour utiliser une propriété de classe dans un template, vous devez utiliser la syntaxe `{{ }}`.
</docs-step>

<docs-step title="Mettre à jour le template du composant {#update-the-component-template}">
Mettez à jour la propriété `template` pour qu'elle corresponde au HTML suivant :

```ts
template: `Hello {{ city }}`,
```

C'est un exemple d'interpolation, qui fait partie de la syntaxe de template d'Angular. Elle vous permet de faire bien plus que placer du texte dynamique dans un template. Vous pouvez aussi utiliser cette syntaxe pour appeler des fonctions, écrire des expressions et plus encore.
</docs-step>

<docs-step title="S'entraîner davantage avec l'interpolation {#more-practice-with-interpolation}">
Essayez ceci : ajoutez une autre paire de `{{ }}` contenant `1 + 1` :

```ts
template: `Hello {{ city }}, {{ 1 + 1 }}`,
```

Angular évalue le contenu des `{{ }}` et affiche le résultat dans le template.
</docs-step>

</docs-workflow>

Ce n'est qu'un aperçu de ce qu'il est possible de faire avec les templates Angular ; continuez votre apprentissage pour en découvrir davantage.
