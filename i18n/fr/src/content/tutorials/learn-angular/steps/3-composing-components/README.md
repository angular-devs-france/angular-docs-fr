<!-- source: eb67d0fb950074d006859bfc8c04d27d50bc5be6 -->
# Composer des composants

Vous avez appris à mettre à jour le template, la logique et les styles d'un composant, mais comment utiliser un composant dans votre application ?

La propriété `selector` de la configuration du composant vous donne un nom à utiliser pour référencer le composant dans un autre template. Vous utilisez le `selector` comme une balise HTML : par exemple, `app-user` devient `<app-user />` dans le template.

NOTE: Pour en savoir plus, consultez [l'utilisation des composants dans le guide des Fondamentaux](/essentials/components#using-components).

Dans cette activité, vous allez apprendre à composer des composants.

<hr/>

Dans cet exemple, il y a deux composants, `User` et `App`.

<docs-workflow>

<docs-step title="Ajouter une référence à `User` {#add-a-reference-to-user}">
Mettez à jour le template de `App` pour y inclure une référence à `User`, qui utilise le selector `app-user`. Pensez à ajouter `User` au tableau imports de `App` : cela le rend utilisable dans le template de `App`.

```angular-html
template: `<app-user />`, imports: [User]
```

Le composant affiche maintenant le message `Username: youngTech`. Vous pouvez mettre à jour le code du template pour y ajouter davantage de balisage.
</docs-step>

<docs-step title="Ajouter davantage de balisage {#add-more-markup}">
Comme vous pouvez utiliser n'importe quel balisage HTML dans un template, essayez de mettre à jour le template de `App` pour qu'il contienne aussi d'autres éléments HTML. Cet exemple ajoute un élément `<section>` comme parent de l'élément `<app-user>`.

```angular-html
template: `
<section><app-user /></section>
`,
```

</docs-step>

</docs-workflow>
Vous pouvez utiliser autant de balisage HTML et de composants que nécessaire pour concrétiser votre idée d'application. Vous pouvez même avoir plusieurs exemplaires de votre composant sur la même page.

Cela nous amène à une bonne question : comment afficher un composant de façon conditionnelle en fonction des données ? Passez à la section suivante pour le découvrir.
