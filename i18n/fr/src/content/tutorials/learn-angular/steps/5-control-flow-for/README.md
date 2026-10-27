<!-- source: 17fe4a159a5d67ffa382ddb990452b214e37b6cf -->
# Le control flow dans les composants : `@for`

Lorsque vous créez des applications web, vous devez souvent répéter du code un certain nombre de fois : par exemple, à partir d'un tableau de noms, vous voudrez peut-être afficher chaque nom dans une balise `<p>`.

NOTE: Pour en savoir plus, consultez [le control flow dans le guide des Fondamentaux](/essentials/templates#control-flow-with-if-and-for).

Dans cette activité, vous allez apprendre à utiliser `@for` pour répéter des éléments dans un template.

<hr/>

La syntaxe qui permet de répéter des éléments dans un template est `@for`.

Voici un exemple d'utilisation de la syntaxe `@for` dans un composant :

```angular-ts
@Component({
  ...
  template: `
    @for (os of operatingSystems; track os.id) {
      {{ os.name }}
    }
  `,
})
export class App {
  operatingSystems = [{id: 'win', name: 'Windows'}, {id: 'osx', name: 'macOS'}, {id: 'linux', name: 'Linux'}];
}
```

Deux points à noter :

- Le `for` est précédé d'un `@`, car il s'agit d'une syntaxe particulière appelée [syntaxe de template d'Angular](guide/templates)
- Pour les applications qui utilisent encore `*ngFor`, consultez la [documentation Angular sur NgFor](guide/directives/structural-directives)

<docs-workflow>

<docs-step title="Ajouter la propriété `users` {#add-the-users-property}">
Dans la classe `App`, ajoutez une propriété nommée `users` qui contient des utilisateurs et leurs noms.

```ts
[
  {id: 0, name: 'Sarah'},
  {id: 1, name: 'Amy'},
  {id: 2, name: 'Rachel'},
  {id: 3, name: 'Jessica'},
  {id: 4, name: 'Poornima'},
];
```

</docs-step>

<docs-step title="Mettre à jour le template {#update-the-template}">
Mettez à jour le template pour afficher le nom de chaque utilisateur dans un élément `p` à l'aide de la syntaxe de template `@for`.

```angular-html
@for (user of users; track user.id) {
  <p>{{ user.name }}</p>
}
```

NOTE: l'utilisation de `track` est obligatoire ; vous pouvez utiliser l'`id` ou un autre identifiant unique.

</docs-step>

</docs-workflow>

Ce type de fonctionnalité s'appelle le control flow. Vous allez ensuite apprendre à personnaliser les composants et à communiquer avec eux. Au passage, vous faites du très bon travail jusqu'ici.
