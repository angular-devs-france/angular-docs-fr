<!-- source: e1c61c301d2a9fbb558a8970be49887f2b60acb8 -->
# Valider les saisies de formulaire

Vous pouvez améliorer la qualité globale des données en validant l'exactitude et la complétude des saisies de l'utilisateur.
Cette page montre comment valider les saisies de l'utilisateur depuis l'interface et afficher des messages de validation utiles, aussi bien dans les reactive forms que dans les template-driven forms.

## Valider les saisies dans les template-driven forms {#validating-input-in-template-driven-forms}

Pour ajouter de la validation à un template-driven form, ajoutez les mêmes attributs de validation que pour la [validation native des formulaires HTML](https://developer.mozilla.org/docs/Web/Guide/HTML/HTML5/Constraint_validation).
Angular utilise des directives pour faire correspondre ces attributs aux fonctions de validation du framework.

Chaque fois que la valeur d'un contrôle de formulaire change, Angular exécute la validation et génère soit une liste d'erreurs de validation qui aboutit à un statut `INVALID`, soit `null`, qui aboutit à un statut `VALID`.

Vous pouvez ensuite inspecter l'état du contrôle en exportant `ngModel` dans une variable de template locale.
L'exemple suivant exporte `NgModel` dans une variable nommée `name` :

<docs-code header="actor-form-template.component.html (name)" path="adev/src/content/examples/form-validation/src/app/template/actor-form-template.component.html" region="name-with-error-msg"/>

Remarquez les fonctionnalités suivantes illustrées par l'exemple.

- L'élément `<input>` porte les attributs de validation HTML : `required` et `minlength`.
  Il porte aussi une directive de validateur personnalisée, `forbiddenName`.
  Pour plus d'informations, consultez la section [Validateurs personnalisés](#defining-custom-validators).

- `#name="ngModel"` exporte `NgModel` dans une variable locale nommée `name`.
  `NgModel` reflète de nombreuses propriétés de son instance de `FormControl` sous-jacente : vous pouvez donc l'utiliser dans le template pour vérifier les états du contrôle comme `valid` et `dirty`.
  Pour la liste complète des propriétés de contrôle, consultez la référence d'API [AbstractControl](api/forms/AbstractControl).
  - Le `@if` le plus externe affiche un ensemble de messages imbriqués, mais seulement si `name` est invalide et que le contrôle est soit `dirty`, soit `touched`.

  - Chaque `@if` imbriqué peut présenter un message personnalisé pour l'une des erreurs de validation possibles.
    Il y a des messages pour `required`, `minlength` et `forbiddenName`.

HELPFUL: Pour éviter que le validateur n'affiche des erreurs avant que l'utilisateur ait eu l'occasion de modifier le formulaire, vérifiez l'état `dirty` ou `touched` du contrôle.

- Quand l'utilisateur modifie la valeur du champ observé, le contrôle est marqué comme modifié (dirty)
- Quand l'élément du contrôle de formulaire perd le focus, le contrôle est marqué comme touché (touched)

## Valider les saisies dans les reactive forms {#validating-input-in-reactive-forms}

Dans un reactive form, la source de vérité est la classe du composant.
Au lieu d'ajouter des validateurs via des attributs dans le template, vous ajoutez des fonctions de validation directement au model du contrôle de formulaire dans la classe du composant.
Angular appelle ensuite ces fonctions chaque fois que la valeur du contrôle change.

### Fonctions de validation {#validator-functions}

Les fonctions de validation peuvent être synchrones ou asynchrones.

| Type de validateur   | Détails |
| :--------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Validateurs synchrones  | Fonctions synchrones qui prennent une instance de contrôle et renvoient immédiatement soit un ensemble d'erreurs de validation, soit `null`. Passez-les comme deuxième argument lorsque vous instanciez un `FormControl`. |
| Validateurs asynchrones | Fonctions asynchrones qui prennent une instance de contrôle et renvoient une Promise ou un Observable qui émet plus tard un ensemble d'erreurs de validation ou `null`. Passez-les comme troisième argument lorsque vous instanciez un `FormControl`. |

Pour des raisons de performances, Angular n'exécute les validateurs asynchrones que si tous les validateurs synchrones réussissent.
Chacun doit se terminer avant que les erreurs ne soient définies.

### Fonctions de validation intégrées {#built-in-validator-functions}

Vous pouvez choisir d'[écrire vos propres fonctions de validation](#defining-custom-validators), ou utiliser certains des validateurs intégrés d'Angular.

Les validateurs intégrés disponibles sous forme d'attributs dans les template-driven forms, comme `required` et `minlength`, sont tous disponibles sous forme de fonctions dans la classe `Validators`.
Pour la liste complète des validateurs intégrés, consultez la référence d'API [Validators](api/forms/Validators).

Pour transformer le formulaire d'acteur en reactive form, utilisez certains de ces mêmes
validateurs intégrés, cette fois sous forme de fonctions, comme dans l'exemple suivant.

<docs-code header="actor-form-reactive.component.ts (fonctions de validation)" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.1.ts" region="form-group"/>

Dans cet exemple, le contrôle `name` met en place deux validateurs intégrés, `Validators.required` et `Validators.minLength(4)`, et un validateur personnalisé, `forbiddenNameValidator`.

Tous ces validateurs sont synchrones : ils sont donc passés comme deuxième argument.
Remarquez que vous pouvez prendre en charge plusieurs validateurs en passant les fonctions sous forme de tableau.

Cet exemple ajoute aussi quelques méthodes getter.
Dans un reactive form, vous pouvez toujours accéder à n'importe quel contrôle de formulaire via la méthode `get` de son groupe parent, mais il est parfois utile de définir des getters comme raccourcis pour le template.

Si vous regardez à nouveau le template de l'input `name`, il est assez similaire à l'exemple template-driven.

<docs-code header="actor-form-reactive.component.html (name avec message d'erreur)" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.html" region="name-with-error-msg"/>

Ce formulaire diffère de la version template-driven en ce qu'il n'exporte plus aucune directive. À la place, il utilise le getter `name` défini dans la classe du composant.

Remarquez que l'attribut `required` est toujours présent dans le template. Bien qu'il ne soit pas nécessaire à la validation, il doit être conservé pour des raisons d'accessibilité.

## Définir des validateurs personnalisés {#defining-custom-validators}

Les validateurs intégrés ne correspondent pas toujours exactement au cas d'usage de votre application : vous devez donc parfois créer un validateur personnalisé.

Prenez la fonction `forbiddenNameValidator` de l'exemple précédent.
Voici à quoi ressemble la définition de cette fonction.

<docs-code header="forbidden-name.directive.ts (forbiddenNameValidator)" path="adev/src/content/examples/form-validation/src/app/shared/forbidden-name.directive.ts" region="custom-validator"/>

La fonction est une fabrique qui prend une expression régulière pour détecter un nom interdit _spécifique_ et renvoie une fonction de validation.

Dans cet exemple, le nom interdit est « bob » : le validateur rejette donc tout nom d'acteur contenant « bob ».
Ailleurs, il pourrait rejeter « alice » ou tout nom correspondant à l'expression régulière de configuration.

La fabrique `forbiddenNameValidator` renvoie la fonction de validation configurée.
Cette fonction prend un objet contrôle Angular et renvoie _soit_ null si la valeur du contrôle est valide, _soit_ un objet d'erreur de validation.
L'objet d'erreur de validation a généralement une propriété dont le nom est la clé de validation, `'forbiddenName'`, et dont la valeur est un dictionnaire arbitraire de valeurs que vous pourriez insérer dans un message d'erreur, `{name}`.

Les validateurs asynchrones personnalisés sont similaires aux validateurs synchrones, mais ils doivent renvoyer une promise ou un observable qui émet plus tard null ou un objet d'erreur de validation.
Dans le cas d'un observable, celui-ci doit se terminer : le formulaire utilise alors la dernière valeur émise pour la validation.

### Ajouter des validateurs personnalisés aux reactive forms {#adding-custom-validators-to-reactive-forms}

Dans les reactive forms, ajoutez un validateur personnalisé en passant la fonction directement au `FormControl`.

<docs-code header="actor-form-reactive.component.ts (fonctions de validation)" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.1.ts" region="custom-validator"/>

### Ajouter des validateurs personnalisés aux template-driven forms {#adding-custom-validators-to-template-driven-forms}

Dans les template-driven forms, ajoutez au template une directive qui encapsule la fonction de validation.
Par exemple, la directive `ForbiddenValidatorDirective` correspondante sert d'enveloppe autour de `forbiddenNameValidator`.

Angular reconnaît le rôle de la directive dans le processus de validation, car la directive s'enregistre auprès du provider `NG_VALIDATORS`, comme le montre l'exemple suivant.
`NG_VALIDATORS` est un provider prédéfini avec une collection extensible de validateurs.

<docs-code header="forbidden-name.directive.ts (providers)" path="adev/src/content/examples/form-validation/src/app/shared/forbidden-name.directive.ts" region="directive-providers"/>

La classe de la directive implémente ensuite l'interface `Validator`, afin de s'intégrer facilement aux formulaires Angular.
Voici le reste de la directive, pour vous donner une idée de la façon dont l'ensemble s'articule.

<docs-code header="forbidden-name.directive.ts (directive)" path="adev/src/content/examples/form-validation/src/app/shared/forbidden-name.directive.ts" region="directive"/>

Une fois la `ForbiddenValidatorDirective` prête, vous pouvez ajouter son selector, `appForbiddenName`, à n'importe quel élément de saisie pour l'activer.
Par exemple :

<docs-code header="actor-form-template.component.html (forbidden-name-input)" path="adev/src/content/examples/form-validation/src/app/template/actor-form-template.component.html" region="name-input"/>

HELPFUL: Remarquez que la directive de validation personnalisée est instanciée avec `useExisting` plutôt qu'avec `useClass`.
Le validateur enregistré doit être _cette instance_ de la `ForbiddenValidatorDirective`, c'est-à-dire l'instance présente dans le formulaire dont la propriété `forbiddenName` est liée à « bob ».

Si vous remplaciez `useExisting` par `useClass`, vous enregistreriez une nouvelle instance de classe, qui n'a pas de `forbiddenName`.

## Classes CSS de statut des contrôles {#control-status-css-classes}

Angular reflète automatiquement de nombreuses propriétés des contrôles sur l'élément du contrôle de formulaire sous forme de classes CSS.
Utilisez ces classes pour styliser les éléments des contrôles de formulaire selon l'état du formulaire.
Les classes suivantes sont actuellement prises en charge.

- `.ng-valid`
- `.ng-invalid`
- `.ng-pending`
- `.ng-pristine`
- `.ng-dirty`
- `.ng-untouched`
- `.ng-touched`
- `.ng-submitted` \(uniquement sur l'élément de formulaire englobant\)

Dans l'exemple suivant, le formulaire d'acteur utilise les classes `.ng-valid` et `.ng-invalid` pour
définir la couleur de la bordure de chaque contrôle de formulaire.

<docs-code header="forms.css (classes de statut)" path="adev/src/content/examples/form-validation/src/assets/forms.css"/>

## Validation inter-champs {#cross-field-validation}

Un validateur inter-champs est un [validateur personnalisé](#defining-custom-validators 'Read about custom validators') qui compare les valeurs de différents champs d'un formulaire et les accepte ou les rejette en combinaison.
Par exemple, vous pouvez avoir un formulaire qui propose des options mutuellement incompatibles : l'utilisateur peut choisir A ou B, mais pas les deux.
Certaines valeurs de champs peuvent aussi dépendre d'autres ; un utilisateur pourrait n'être autorisé à choisir B que si A est également choisi.

Les exemples de validation inter-champs suivants montrent comment :

- Valider les saisies d'un reactive form ou d'un template-driven form à partir des valeurs de deux contrôles frères,
- Afficher un message d'erreur descriptif une fois que l'utilisateur a interagi avec le formulaire et que la validation a échoué.

Les exemples utilisent la validation inter-champs pour s'assurer que les acteurs ne réutilisent pas leur propre nom dans leur rôle en remplissant l'Actor Form.
Les validateurs y parviennent en vérifiant que les noms des acteurs et les rôles ne correspondent pas.

### Ajouter la validation inter-champs aux reactive forms {#adding-cross-validation-to-reactive-forms}

Le formulaire a la structure suivante :

```ts
const actorForm = new FormGroup({
  'name': new FormControl(),
  'role': new FormControl(),
  'skill': new FormControl(),
});
```

Remarquez que `name` et `role` sont des contrôles frères.
Pour évaluer les deux contrôles dans un seul validateur personnalisé, vous devez effectuer la validation dans un contrôle ancêtre commun : le `FormGroup`.
Vous interrogez le `FormGroup` pour obtenir ses contrôles enfants afin de pouvoir comparer leurs valeurs.

Pour ajouter un validateur au `FormGroup`, passez le nouveau validateur comme deuxième argument lors de sa création.

```ts
const actorForm = new FormGroup(
  {
    'name': new FormControl(),
    'role': new FormControl(),
    'skill': new FormControl(),
  },
  {validators: unambiguousRoleValidator},
);
```

Le code du validateur est le suivant.

<docs-code header="unambiguous-role.directive.ts" path="adev/src/content/examples/form-validation/src/app/shared/unambiguous-role.directive.ts" region="cross-validation-validator"/>

Le validateur `unambiguousRoleValidator` implémente l'interface `ValidatorFn`.
Il prend un objet contrôle Angular en argument et renvoie soit null si le formulaire est valide, soit `ValidationErrors` dans le cas contraire.

Le validateur récupère les contrôles enfants en appelant la méthode [get](api/forms/AbstractControl#get) du `FormGroup`, puis compare les valeurs des contrôles `name` et `role`.

Si les valeurs ne correspondent pas, le rôle est sans ambiguïté, les deux sont valides et le validateur renvoie null.
Si elles correspondent, le rôle de l'acteur est ambigu et le validateur doit marquer le formulaire comme invalide en renvoyant un objet d'erreur.

Pour offrir une meilleure expérience utilisateur, le template affiche un message d'erreur approprié lorsque le formulaire est invalide.

<docs-code header="actor-form-reactive.component.html" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.html" region="cross-validation-error-message"/>

Ce `@if` affiche l'erreur si le `FormGroup` contient l'erreur de validation inter-champs renvoyée par le validateur `unambiguousRoleValidator`, mais seulement si l'utilisateur a fini d'[interagir avec le formulaire](#control-status-css-classes).

### Ajouter la validation inter-champs aux template-driven forms {#adding-cross-validation-to-template-driven-forms}

Pour un template-driven form, vous devez créer une directive qui encapsule la fonction de validation.
Vous fournissez cette directive comme validateur à l'aide du [jeton `NG_VALIDATORS`](/api/forms/NG_VALIDATORS), comme le montre l'exemple suivant.

<docs-code header="unambiguous-role.directive.ts" path="adev/src/content/examples/form-validation/src/app/shared/unambiguous-role.directive.ts" region="cross-validation-directive"/>

Vous devez ajouter la nouvelle directive au template HTML.
Comme le validateur doit être enregistré au niveau le plus élevé du formulaire, le template suivant place la directive sur la balise `form`.

<docs-code header="actor-form-template.component.html" path="adev/src/content/examples/form-validation/src/app/template/actor-form-template.component.html" region="cross-validation-register-validator"/>

Pour offrir une meilleure expérience utilisateur, un message d'erreur approprié s'affiche lorsque le formulaire est invalide.

<docs-code header="actor-form-template.component.html" path="adev/src/content/examples/form-validation/src/app/template/actor-form-template.component.html" region="cross-validation-error-message"/>

C'est identique dans les template-driven forms et les reactive forms.

## Créer des validateurs asynchrones {#creating-asynchronous-validators}

Les validateurs asynchrones implémentent les interfaces `AsyncValidatorFn` et `AsyncValidator`.
Ils sont très similaires à leurs équivalents synchrones, avec les différences suivantes.

- Les fonctions `validate()` doivent renvoyer une promise ou un observable,
- L'observable renvoyé doit être fini, c'est-à-dire qu'il doit se terminer à un moment donné.
  Pour convertir un observable infini en observable fini, faites passer l'observable par un opérateur de filtrage comme `first`, `last`, `take` ou `takeUntil`.

La validation asynchrone a lieu après la validation synchrone, et n'est effectuée que si la validation synchrone réussit.
Cette vérification permet aux formulaires d'éviter des processus de validation asynchrone potentiellement coûteux \(comme une requête HTTP\) si les méthodes de validation plus basiques ont déjà détecté une saisie invalide.

Une fois la validation asynchrone commencée, le contrôle de formulaire passe dans un état `pending`.
Inspectez la propriété `pending` du contrôle et utilisez-la pour donner un retour visuel sur l'opération de validation en cours.

Un pattern d'interface courant consiste à afficher un spinner pendant l'exécution de la validation asynchrone.
L'exemple suivant montre comment y parvenir dans un template-driven form.

```angular-html
<input [(ngModel)]="name" #model="ngModel" appSomeAsyncValidator />

@if (model.pending) {
  <app-spinner />
}
```

### Implémenter un validateur asynchrone personnalisé {#implementing-a-custom-async-validator}

Dans l'exemple suivant, un validateur asynchrone garantit que les acteurs sont choisis pour un rôle qui n'est pas déjà attribué.
De nouveaux acteurs passent constamment des auditions et d'anciens acteurs prennent leur retraite : la liste des rôles disponibles ne peut donc pas être récupérée à l'avance.
Pour valider le rôle saisi, le validateur doit lancer une opération asynchrone pour consulter une base de données centrale de tous les acteurs actuellement engagés.

Le code suivant crée la classe du validateur, `UniqueRoleValidator`, qui implémente l'interface `AsyncValidator`.

<docs-code header="role.directive.ts" path="adev/src/content/examples/form-validation/src/app/shared/role.directive.ts" region="async-validator"/>

La propriété `actorsService` est initialisée avec une instance du jeton `ActorsService`, qui définit l'interface suivante.

```ts
interface ActorsService {
  isRoleTaken: (role: string) => Observable<boolean>;
}
```

Dans une application réelle, `ActorsService` serait chargé d'effectuer une requête HTTP vers la base de données des acteurs pour vérifier si le rôle est disponible.
Du point de vue du validateur, l'implémentation réelle du service n'a pas d'importance : l'exemple peut donc simplement s'appuyer sur l'interface `ActorsService`.

Au début de la validation, `UniqueRoleValidator` délègue à la méthode `isRoleTaken()` d'`ActorsService` avec la valeur actuelle du contrôle.
À ce stade, le contrôle est marqué comme `pending` et reste dans cet état jusqu'à ce que la chaîne d'observables renvoyée par la méthode `validate()` se termine.

La méthode `isRoleTaken()` envoie une requête HTTP qui vérifie si le rôle est disponible, et renvoie `Observable<boolean>` comme résultat.
La méthode `validate()` fait passer la réponse par l'opérateur `map` et la transforme en résultat de validation.

La méthode, comme tout validateur, renvoie ensuite `null` si le formulaire est valide, et `ValidationErrors` dans le cas contraire.
Ce validateur gère les erreurs éventuelles avec l'opérateur `catchError`.
Dans ce cas, le validateur traite l'erreur de `isRoleTaken()` comme une validation réussie, car l'échec d'une requête de validation ne signifie pas nécessairement que le rôle est invalide.
Vous pourriez gérer l'erreur différemment et renvoyer plutôt l'objet `ValidationError`.

Au bout d'un certain temps, la chaîne d'observables se termine et la validation asynchrone est achevée.
L'indicateur `pending` passe à `false`, et la validité du formulaire est mise à jour.

### Ajouter des validateurs asynchrones aux reactive forms {#adding-async-validators-to-reactive-forms}

Pour utiliser un validateur asynchrone dans les reactive forms, commencez par injecter le validateur dans une propriété de la classe du composant.

<docs-code header="actor-form-reactive.component.ts" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.2.ts" region="async-validator-inject"/>

Ensuite, passez la fonction de validation directement au `FormControl` pour l'appliquer.

Dans l'exemple suivant, la fonction `validate` de `UniqueRoleValidator` est appliquée à `roleControl` en la passant à l'option `asyncValidators` du contrôle et en la liant à l'instance de `UniqueRoleValidator` injectée dans `ActorFormReactiveComponent`.
La valeur de `asyncValidators` peut être soit une seule fonction de validation asynchrone, soit un tableau de fonctions.
Pour en savoir plus sur les options de `FormControl`, consultez la référence d'API [AbstractControlOptions](api/forms/AbstractControlOptions).

<docs-code header="actor-form-reactive.component.ts" path="adev/src/content/examples/form-validation/src/app/reactive/actor-form-reactive.component.2.ts" region="async-validator-usage"/>

### Ajouter des validateurs asynchrones aux template-driven forms {#adding-async-validators-to-template-driven-forms}

Pour utiliser un validateur asynchrone dans les template-driven forms, créez une nouvelle directive et enregistrez-y le provider `NG_ASYNC_VALIDATORS`.

Dans l'exemple ci-dessous, la directive injecte la classe `UniqueRoleValidator`, qui contient la logique de validation proprement dite, et l'invoque dans la fonction `validate`, déclenchée par Angular lorsque la validation doit avoir lieu.

<docs-code header="role.directive.ts" path="adev/src/content/examples/form-validation/src/app/shared/role.directive.ts" region="async-validator-directive"/>

Ensuite, comme pour les validateurs synchrones, ajoutez le selector de la directive à un élément de saisie pour l'activer.

<docs-code header="actor-form-template.component.html (unique-unambiguous-role-input)" path="adev/src/content/examples/form-validation/src/app/template/actor-form-template.component.html" region="role-input"/>

### Optimiser les performances des validateurs asynchrones {#optimizing-performance-of-async-validators}

Par défaut, tous les validateurs s'exécutent après chaque changement de valeur du formulaire.
Avec les validateurs synchrones, cela n'a normalement pas d'impact notable sur les performances de l'application.
Les validateurs asynchrones, en revanche, effectuent généralement une sorte de requête HTTP pour valider le contrôle.
Envoyer une requête HTTP après chaque frappe au clavier pourrait surcharger l'API backend, et doit être évité si possible.

Vous pouvez retarder la mise à jour de la validité du formulaire en changeant la propriété `updateOn` de `change` (par défaut) à `submit` ou `blur`.

Avec les template-driven forms, définissez la propriété dans le template.

```angular-html
<input [(ngModel)]="name" [ngModelOptions]="{updateOn: 'blur'}" />
```

Avec les reactive forms, définissez la propriété dans l'instance de `FormControl`.

```ts
new FormControl('', {updateOn: 'blur'});
```

## Gérer les validateurs dynamiquement dans les reactive forms {#managing-validators-dynamically-in-reactive-forms}

Dans les reactive forms complexes, vous pouvez avoir besoin d'ajouter, de supprimer ou de modifier des validateurs en fonction des saisies de l'utilisateur ou de l'état de l'application.
Angular fournit plusieurs méthodes sur `AbstractControl` pour gérer les validateurs à l'exécution sans recréer les contrôles de formulaire.

### Ajouter et supprimer des validateurs {#adding-and-removing-validators}

Les méthodes [`addValidators`](api/forms/AbstractControl#addValidators) et [`removeValidators`](api/forms/AbstractControl#removeValidators) vous permettent de modifier les validateurs d'un contrôle après son initialisation.

```ts
onCountryChange(country: string) {
    const postalCodeControl = this.profileForm.get('postalCode');

    if (country === 'US') {
      // Add validators for US postal codes
      postalCodeControl.addValidators([Validators.required, Validators.pattern(/^\d{5}$/)]);
    } else {
      // Remove validators when not US
      postalCodeControl.removeValidators([Validators.required]);
    }

    postalCodeControl.updateValueAndValidity();
}
```

### Remplacer tous les validateurs {#replacing-all-validators}

Utilisez [`setValidators`](api/forms/AbstractControl#setValidators) pour remplacer tous les validateurs synchrones existants d'un contrôle, ou [`clearValidators`](api/forms/AbstractControl#clearValidators) pour supprimer tous les validateurs.

```ts
toggleStrictNameValidation(isStrict: boolean) {
  const nameControl = this.profileForm.get('name');

  if (isStrict) {
    // Set strict validation rules
    nameControl.setValidators([
      Validators.required,
      Validators.minLength(3),
      Validators.pattern(/^[a-zA-Z]+$/),
    ]);
  } else {
    // Clear all validators
    nameControl.clearValidators();
  }

  nameControl.updateValueAndValidity();
}
```

Le même pattern s'applique aux validateurs asynchrones avec [`addAsyncValidators`](api/forms/AbstractControl#addAsyncValidators), [`removeAsyncValidators`](api/forms/AbstractControl#removeAsyncValidators), [`setAsyncValidators`](api/forms/AbstractControl#setAsyncValidators) et [`clearAsyncValidators`](api/forms/AbstractControl#clearAsyncValidators).

### Déclencher les mises à jour de la validation {#triggering-validation-updates}

Après avoir modifié les validateurs, appelez [`updateValueAndValidity`](api/forms/AbstractControl#updateValueAndValidity) pour recalculer le statut de validation du contrôle.
Cette méthode accepte des options pour contrôler le comportement de la mise à jour.

```ts
// Update control and notify parent
control.updateValueAndValidity();

// Update control only, don't notify parent or emit events
control.updateValueAndValidity({onlySelf: true, emitEvent: false});
```

## Interaction avec la validation native des formulaires HTML {#interaction-with-native-html-form-validation}

Par défaut, Angular désactive la [validation native des formulaires HTML](https://developer.mozilla.org/docs/Web/Guide/HTML/Constraint_validation) en ajoutant l'attribut `novalidate` sur le `<form>` englobant, et utilise des directives pour faire correspondre ces attributs aux fonctions de validation du framework.
Si vous voulez utiliser la validation native **en combinaison** avec la validation d'Angular, vous pouvez la réactiver avec la directive `ngNativeValidate`.
Consultez la [documentation de l'API](api/forms/NgForm#native-dom-validation-ui) pour plus de détails.
