<!-- source: 79a2bb39357e303cc7b250efbfc0f47a85cfcc56 -->
# Présentation des formulaires

Les formulaires occupent une place importante dans de nombreuses applications, car ils permettent à votre application de recevoir les saisies de l'utilisateur. Voyons comment Angular gère les formulaires.

Angular propose deux types de formulaires : les template-driven forms et les reactive forms. Vous allez découvrir les deux au fil des prochaines activités.

NOTE: Pour en savoir plus, consultez [les formulaires dans Angular dans le guide approfondi](/guide/forms).

Dans cette activité, vous allez apprendre à mettre en place un formulaire avec l'approche template-driven.

<hr>

<docs-workflow>

<docs-step title="Créer un champ de saisie {#create-an-input-field}">

Dans `user.ts`, mettez à jour le template en ajoutant un champ de saisie de texte dont l'`id` vaut `framework` et le type vaut `text`.

```angular-html
<label for="framework">
  Favorite Framework:
  <input id="framework" type="text" />
</label>
```

</docs-step>

<docs-step title="Importer `FormsModule` {#import-formsmodule}">

Pour que ce formulaire utilise les fonctionnalités d'Angular qui permettent le data binding avec les formulaires, vous devez importer le `FormsModule`.

Importez le `FormsModule` depuis `@angular/forms` et ajoutez-le au tableau `imports` de `User`.

```ts {highlight:[[2],[6]]}
import {Component} from '@angular/core';
import {FormsModule} from '@angular/forms';

@Component({
...
imports: [FormsModule],
})
export class User {}
```

</docs-step>

<docs-step title="Ajouter un binding sur la valeur du champ {#add-binding-to-the-value-of-the-input}">

Le `FormsModule` fournit une directive appelée `ngModel`, qui lie la valeur du champ à une propriété de votre classe.

Mettez à jour le champ pour utiliser la directive `ngModel`, précisément avec la syntaxe `[(ngModel)]="favoriteFramework"`, afin de le lier à la propriété `favoriteFramework`.

```html {highlight:[3]}
<label for="framework">
  Favorite Framework:
  <input id="framework" type="text" [(ngModel)]="favoriteFramework" />
</label>
```

Une fois les modifications faites, essayez de saisir une valeur dans le champ. Remarquez comment elle se met à jour à l'écran (oui, c'est très cool).

NOTE: La syntaxe `[()]` est surnommée « banana in a box », mais elle représente le two-way binding : un property binding et un event binding. Pour en savoir plus, consultez la [documentation Angular sur le two-way binding](guide/templates/two-way-binding).

</docs-step>

</docs-workflow>

Vous venez de franchir une première étape importante vers la création de formulaires avec Angular.

Beau travail. Gardons le rythme !
