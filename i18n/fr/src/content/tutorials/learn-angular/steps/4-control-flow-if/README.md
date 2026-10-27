<!-- source: 36cc9b079a97c2730af2bd7da134c206bd6a5f17 -->
# Le control flow dans les composants : `@if`

Décider de ce qu'il faut afficher à l'écran pour un utilisateur est une tâche courante en développement d'applications. Bien souvent, cette décision est prise par programmation à l'aide de conditions.

Pour exprimer des affichages conditionnels dans les templates, Angular utilise la syntaxe de template `@if`.

NOTE: Pour en savoir plus, consultez [le control flow dans le guide des Fondamentaux](/essentials/templates#control-flow-with-if-and-for).

Dans cette activité, vous allez apprendre à utiliser des conditions dans les templates.

<hr/>

La syntaxe qui permet l'affichage conditionnel d'éléments dans un template est `@if`.

Voici un exemple d'utilisation de la syntaxe `@if` dans un composant :

```angular-ts
@Component({
  ...
  template: `
    @if (isLoggedIn) {
      <p>Welcome back, Friend!</p>
    }
  `,
})
export class App {
  isLoggedIn = true;
}
```

Deux points à noter :

- Le `if` est précédé d'un `@`, car il s'agit d'un type de syntaxe particulier appelé [syntaxe de template d'Angular](guide/templates)
- Pour les applications qui utilisent encore `*ngIf`, consultez la [documentation Angular sur NgIf](guide/directives/structural-directives) pour plus d'informations.

<docs-workflow>

<docs-step title="Créer une propriété nommée `isServerRunning` {#create-a-property-called-isserverrunning}">
Dans la classe `App`, ajoutez une propriété `boolean` nommée `isServerRunning` et définissez sa valeur initiale à `true`.
</docs-step>

<docs-step title="Utiliser `@if` dans le template {#use-if-in-the-template}">
Mettez à jour le template pour afficher le message `Yes, the server is running` si la valeur de `isServerRunning` est `true`.

</docs-step>

<docs-step title="Utiliser `@else` dans le template {#use-else-in-the-template}">
Angular prend désormais en charge une syntaxe de template native pour définir le cas « sinon » avec la syntaxe `@else`. Mettez à jour le template pour afficher le message `No, the server is not running` dans le cas « sinon ».

Voici un exemple :

```angular-html
template: `
@if (isServerRunning) {
  ...
} @else {
  ...
}
`;
```

Ajoutez votre code pour compléter le balisage manquant.

</docs-step>

</docs-workflow>

Ce type de fonctionnalité s'appelle le control flow conditionnel. Vous allez ensuite apprendre à répéter des éléments dans un template.
