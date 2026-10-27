<!-- source: 95c387f87065650bf7f8a6f1b051447a395cd22e -->
# Récupérer la valeur d'un contrôle de formulaire

Maintenant que vos formulaires sont en place avec Angular, l'étape suivante consiste à accéder aux valeurs des contrôles de formulaire.

NOTE: Pour en savoir plus, consultez [l'ajout d'un contrôle de formulaire simple dans le guide approfondi](/guide/forms/reactive-forms#adding-a-basic-form-control).

Dans cette activité, vous allez apprendre à récupérer la valeur du champ de votre formulaire.

<hr>

<docs-workflow>

<docs-step title="Afficher la valeur du champ dans le template {#show-the-value-of-the-input-field-in-the-template}">

Pour afficher la valeur du champ dans un template, vous pouvez utiliser la syntaxe d'interpolation `{{}}`, comme pour n'importe quelle autre propriété de la classe du composant :

```angular-ts {highlight:[5]}
@Component({
  selector: 'app-user',
  template: `
    ...
    <p>Framework: {{ favoriteFramework }}</p>
    <label for="framework">
      Favorite Framework:
      <input id="framework" type="text" [(ngModel)]="favoriteFramework" />
    </label>
  `,
})
export class User {
  favoriteFramework = '';
}
```

</docs-step>

<docs-step title="Récupérer la valeur d'un champ {#retrieve-the-value-of-an-input-field}">

Lorsque vous avez besoin de faire référence à la valeur du champ dans la classe du composant, accédez à la propriété de la classe avec la syntaxe `this`.

```angular-ts {highlight:[15]}
...
@Component({
  selector: 'app-user',
  template: `
    ...
    <button (click)="showFramework()">Show Framework</button>
  `,
  ...
})
export class User {
  favoriteFramework = '';
  ...

  showFramework() {
    alert(this.favoriteFramework);
  }
}
```

</docs-step>

</docs-workflow>

Bravo, vous savez maintenant afficher les valeurs saisies dans votre template et y accéder depuis le code.

Passons à l'autre façon de gérer les formulaires avec Angular : les reactive forms. Pour en savoir plus sur les template-driven forms, consultez [Créer un template-driven form](guide/forms/template-driven-forms).
