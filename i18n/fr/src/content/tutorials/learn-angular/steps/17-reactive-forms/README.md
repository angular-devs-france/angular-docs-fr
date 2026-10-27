<!-- source: d463590cd2149fe45ecb042dc1de695b247029fe -->
# Reactive forms

Lorsque vous voulez gérer vos formulaires depuis le code plutôt que de vous reposer uniquement sur le template, les reactive forms sont la solution.

NOTE: Pour en savoir plus, consultez [les reactive forms dans le guide approfondi](/guide/forms/reactive-forms).

Dans cette activité, vous allez apprendre à mettre en place des reactive forms.

<hr>

<docs-workflow>

<docs-step title="Importer le module `ReactiveForms` {#import-reactiveforms-module}">

Dans `app.ts`, importez `ReactiveFormsModule` depuis `@angular/forms` et ajoutez-le au tableau `imports` du composant.

```angular-ts
import { ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-root',
  template: `
    <form>
      <label>Name
        <input type="text" />
      </label>
      <label>Email
        <input type="email" />
      </label>
      <button type="submit">Submit</button>
    </form>
  `,
  imports: [ReactiveFormsModule],
})
```

</docs-step>

<docs-step title="Créer l'objet `FormGroup` avec des FormControls {#create-the-formgroup-object-with-formcontrols}">

Les reactive forms utilisent la classe `FormControl` pour représenter les contrôles de formulaire (par exemple, les champs de saisie). Angular fournit la classe `FormGroup`, qui regroupe des contrôles de formulaire dans un objet pratique et facilite la gestion des formulaires volumineux pour les développeurs.

Ajoutez `FormControl` et `FormGroup` à l'import depuis `@angular/forms` afin de pouvoir créer un FormGroup pour chaque formulaire, avec les propriétés `name` et `email` en tant que FormControls.

```ts
import {ReactiveFormsModule, FormControl, FormGroup } from '@angular/forms';
...
export class App {
  profileForm = new FormGroup({
    name: new FormControl(''),
    email: new FormControl(''),
  });
}
```

</docs-step>

<docs-step title="Lier le FormGroup et les FormControls au formulaire {#link-the-formgroup-and-formcontrols-to-the-form}">

Chaque `FormGroup` doit être rattaché à un formulaire à l'aide de la directive `[formGroup]`.

De plus, chaque `FormControl` peut être rattaché avec la directive `formControlName` et associé à la propriété correspondante. Mettez à jour le template avec le code de formulaire suivant :

```angular-html
<form [formGroup]="profileForm">
  <label>
    Name
    <input type="text" formControlName="name" />
  </label>
  <label>
    Email
    <input type="email" formControlName="email" />
  </label>
  <button type="submit">Submit</button>
</form>
```

</docs-step>

<docs-step title="Gérer les mises à jour du formulaire {#handle-update-to-the-form}">

Pour accéder aux données du `FormGroup`, il suffit d'accéder à la valeur du `FormGroup`. Mettez à jour le `template` pour afficher les valeurs du formulaire :

```angular-html
...
<h2>Profile Form</h2>
<p>Name: {{ profileForm.value.name }}</p>
<p>Email: {{ profileForm.value.email }}</p>
```

</docs-step>

<docs-step title="Accéder aux valeurs du FormGroup {#access-formgroup-values}">
Ajoutez à la classe du composant une nouvelle méthode appelée `handleSubmit`, que vous utiliserez ensuite pour gérer la soumission du formulaire.
Cette méthode affichera les valeurs du formulaire, auxquelles vous pouvez accéder depuis le FormGroup.

Dans la classe du composant, ajoutez la méthode `handleSubmit()` pour gérer la soumission du formulaire.

```ts
handleSubmit() {
  alert(
    this.profileForm.value.name + ' | ' + this.profileForm.value.email
  );
}
```

</docs-step>

<docs-step title="Ajouter `ngSubmit` au formulaire {#add-ngsubmit-to-the-form}">
Vous avez accès aux valeurs du formulaire ; il est temps de gérer l'événement de soumission et d'utiliser la méthode `handleSubmit`.
Angular fournit un event handler dédié à cet usage, appelé `ngSubmit`. Mettez à jour l'élément form pour appeler la méthode `handleSubmit` lors de la soumission du formulaire.

```angular-html {highlight:[1]}
<form [formGroup]="profileForm" (ngSubmit)="handleSubmit()"></form>
```

</docs-step>

</docs-workflow>

Et voilà, vous savez utiliser les reactive forms dans Angular.

Excellent travail sur cette activité. Continuez pour découvrir la validation des formulaires.
