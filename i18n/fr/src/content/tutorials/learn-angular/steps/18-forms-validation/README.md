<!-- source: 57130011eb65f503cd87c79cf7c45700a2fb9802 -->
# Valider des formulaires

Un autre cas courant lorsque l'on travaille avec des formulaires est la nécessité de valider les saisies, pour s'assurer que les données soumises sont correctes.

NOTE: Pour en savoir plus, consultez [la validation des saisies de formulaire dans le guide approfondi](/guide/forms/reactive-forms#validating-form-input).

Dans cette activité, vous allez apprendre à valider des formulaires avec les reactive forms.

<hr>

<docs-workflow>

<docs-step title="Importer Validators {#import-validators}">

Angular fournit un ensemble d'outils de validation. Pour les utiliser, commencez par mettre à jour le composant pour importer `Validators` depuis `@angular/forms`.

```ts {highlight:[1]}
import {ReactiveFormsModule, Validators} from '@angular/forms';

@Component({...})
export class App {}
```

</docs-step>

<docs-step title="Ajouter une validation au formulaire {#add-validation-to-form}">

Vous pouvez passer à chaque `FormControl` les `Validators` à utiliser pour valider les valeurs du `FormControl`. Par exemple, pour rendre obligatoire le champ `name` de `profileForm`, utilisez `Validators.required`.
Pour le champ `email` de notre formulaire Angular, nous voulons nous assurer qu'il n'est pas laissé vide et qu'il respecte la structure d'une adresse e-mail valide. Pour cela, combinez les validateurs `Validators.required` et `Validators.email` dans un tableau.
Mettez à jour les `FormControl` `name` et `email` :

```ts
profileForm = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', [Validators.required, Validators.email]),
});
```

</docs-step>

<docs-step title="Vérifier la validation du formulaire dans le template {#check-form-validation-in-template}">

Pour déterminer si un formulaire est valide, la classe `FormGroup` dispose d'une propriété `valid`.
Vous pouvez utiliser cette propriété pour lier dynamiquement des attributs. Mettez à jour le `button` de soumission pour qu'il soit activé en fonction de la validité du formulaire.

```angular-html
<button type="submit" [disabled]="!profileForm.valid">Submit</button>
```

</docs-step>

</docs-workflow>

Vous connaissez maintenant les bases du fonctionnement de la validation avec les reactive forms.

Bravo, vous avez découvert ces concepts clés des formulaires dans Angular. Pour aller plus loin, consultez [Valider les saisies de formulaire](guide/forms/form-validation).
