<!-- source: fb6b6f3c8f4e748a54068f35e7c47460c3f1b2bc -->
# Formulaires typés

Depuis Angular 14, les reactive forms sont strictement typés par défaut.

Pour suivre ce guide, vous devez déjà connaître les [reactive forms d'Angular](guide/forms/reactive-forms).

## Présentation des formulaires typés {#overview-of-typed-forms}

<docs-video src="https://www.youtube.com/embed/L-odCf4MfJc" title="Les formulaires typés dans Angular" />

Avec les reactive forms d'Angular, vous spécifiez explicitement un _model de formulaire_. Pour prendre un exemple simple, considérez ce formulaire de connexion basique :

```ts
const login = new FormGroup({
  email: new FormControl(''),
  password: new FormControl(''),
});
```

Angular fournit de nombreuses API pour interagir avec ce `FormGroup`. Par exemple, vous pouvez appeler `login.value`, `login.controls`, `login.patchValue`, etc. (Pour la référence complète de l'API, consultez la [documentation de l'API](api/forms/FormGroup).)

Dans les versions précédentes d'Angular, la plupart de ces API incluaient `any` quelque part dans leurs types, et interagir avec la structure des contrôles, ou avec les valeurs elles-mêmes, n'était pas sûr au niveau du typage. Par exemple, vous pouviez écrire le code invalide suivant :

```ts
const emailDomain = login.value.email.domain;
```

Avec des reactive forms strictement typés, le code ci-dessus ne compile pas, car `email` n'a pas de propriété `domain`.

En plus de cette sûreté supplémentaire, les types apportent diverses autres améliorations, comme une meilleure autocomplétion dans les IDE et un moyen explicite de spécifier la structure du formulaire.

Ces améliorations ne s'appliquent actuellement qu'aux _reactive_ forms (pas aux [_template-driven_ forms](guide/forms/template-driven-forms)).

## Formulaires non typés {#untyped-forms}

Les formulaires non typés sont toujours pris en charge et continueront de fonctionner comme avant. Pour les utiliser, vous devez importer les symboles `Untyped` depuis `@angular/forms` :

```ts
const login = new UntypedFormGroup({
  email: new UntypedFormControl(''),
  password: new UntypedFormControl(''),
});
```

Chaque symbole `Untyped` a exactement la même sémantique que dans les versions précédentes d'Angular. En retirant les préfixes `Untyped`, vous pouvez activer les types de manière progressive.

## `FormControl` : premiers pas {#formcontrol-getting-started}

Le formulaire le plus simple possible se compose d'un seul contrôle :

```ts
const email = new FormControl('angularrox@gmail.com');
```

Le type de ce contrôle sera automatiquement inféré comme `FormControl<string|null>`. TypeScript appliquera automatiquement ce type dans toute l'[API `FormControl`](api/forms/FormControl), par exemple `email.value`, `email.valueChanges` et `email.setValue(...)`.

### Nullabilité {#nullability}

Vous vous demandez peut-être : pourquoi le type de ce contrôle inclut-il `null` ? C'est parce que le contrôle peut devenir `null` à tout moment, en appelant reset :

```ts
const email = new FormControl('angularrox@gmail.com');
email.reset();
console.log(email.value); // null
```

TypeScript vous obligera à toujours gérer la possibilité que le contrôle soit devenu `null`. Si vous voulez rendre ce contrôle non nullable, vous pouvez utiliser l'option `nonNullable`. Le contrôle sera alors réinitialisé à sa valeur initiale, au lieu de `null` :

```ts
const email = new FormControl('angularrox@gmail.com', {nonNullable: true});
email.reset();
console.log(email.value); // angularrox@gmail.com
```

Pour rappel, cette option modifie le comportement de votre formulaire à l'exécution quand `.reset()` est appelé, et doit être activée avec précaution.

### Spécifier un type explicite {#specifying-an-explicit-type}

Il est possible de spécifier le type, au lieu de s'appuyer sur l'inférence. Considérez un contrôle initialisé à `null`. Comme la valeur initiale est `null`, TypeScript inférera `FormControl<null>`, ce qui est plus restreint que ce que nous voulons.

```ts
const email = new FormControl(null);
email.setValue('angularrox@gmail.com'); // Error!
```

Pour éviter cela, nous spécifions explicitement le type `string|null` :

```ts
const email = new FormControl<string | null>(null);
email.setValue('angularrox@gmail.com');
```

## `FormArray` : collections dynamiques et homogènes {#formarray-dynamic-homogenous-collections}

Un `FormArray` contient une liste ouverte de contrôles. Le paramètre de type correspond au type de chaque contrôle interne :

```ts
const names = new FormArray([new FormControl('Alex')]);
names.push(new FormControl('Jess'));
```

Passez un tableau de contrôles à `aliases.push()` quand vous devez ajouter plusieurs entrées à la fois.

```ts
const aliases = new FormArray([new FormControl('ng')]);
aliases.push([new FormControl('ngDev'), new FormControl('ngAwesome')]);
```

Les contrôles internes de ce `FormArray` auront le type `FormControl<string|null>`.

Si vous voulez avoir plusieurs types d'éléments différents dans le tableau, vous devez utiliser `UntypedFormArray`, car TypeScript ne peut pas inférer quel type d'élément se trouvera à quelle position.

Un `FormArray` fournit aussi une méthode `clear()` pour retirer tous les contrôles qu'il contient :

```ts
const aliases = new FormArray([new FormControl('ngDev'), new FormControl('ngAwesome')]);
aliases.clear();
console.log(aliases.length); // 0
```

## `FormGroup` et `FormRecord` {#formgroup-and-formrecord}

Angular fournit le type `FormGroup` pour les formulaires dont l'ensemble de clés est énuméré, et un type appelé `FormRecord` pour les groupes ouverts ou dynamiques.

### Valeurs partielles {#partial-values}

Reprenons un formulaire de connexion :

```ts
const login = new FormGroup({
  email: new FormControl('', {nonNullable: true}),
  password: new FormControl('', {nonNullable: true}),
});
```

Sur n'importe quel `FormGroup`, il est [possible de désactiver des contrôles](api/forms/FormGroup). Un contrôle désactivé n'apparaît pas dans la valeur du groupe.

Par conséquent, le type de `login.value` est `Partial<{email: string, password: string}>`. Le `Partial` de ce type signifie que chaque membre peut être undefined.

Plus précisément, le type de `login.value.email` est `string|undefined`, et TypeScript vous obligera à gérer la valeur potentiellement `undefined` (si vous avez activé `strictNullChecks`).

Si vous voulez accéder à la valeur _en incluant_ les contrôles désactivés, et ainsi éviter les champs potentiellement `undefined`, vous pouvez utiliser `login.getRawValue()`.

### Contrôles optionnels et groupes dynamiques {#optional-controls-and-dynamic-groups}

Certains formulaires ont des contrôles qui peuvent être présents ou non, et qui peuvent être ajoutés et retirés à l'exécution. Vous pouvez représenter ces contrôles avec des _champs optionnels_ :

```ts
interface LoginForm {
  email: FormControl<string>;
  password?: FormControl<string>;
}

const login = new FormGroup<LoginForm>({
  email: new FormControl('', {nonNullable: true}),
  password: new FormControl('', {nonNullable: true}),
});

login.removeControl('password');
```

Dans ce formulaire, nous spécifions explicitement le type, ce qui nous permet de rendre le contrôle `password` optionnel. TypeScript garantira que seuls les contrôles optionnels peuvent être ajoutés ou retirés.

### `FormRecord` {#formrecord}

Certains usages de `FormGroup` ne correspondent pas au schéma ci-dessus, car les clés ne sont pas connues à l'avance. La classe `FormRecord` est conçue pour ce cas :

```ts
const addresses = new FormRecord<FormControl<string | null>>({});
addresses.addControl('Andrew', new FormControl('2340 Folsom St'));
```

N'importe quel contrôle de type `string|null` peut être ajouté à ce `FormRecord`.

Si vous avez besoin d'un `FormGroup` à la fois dynamique (ouvert) et hétérogène (les contrôles sont de types différents), aucune amélioration de la sûreté du typage n'est possible, et vous devez utiliser `UntypedFormGroup`.

Un `FormRecord` peut aussi être construit avec le `FormBuilder` :

```ts
const addresses = fb.record({'Andrew': '2340 Folsom St'});
```

## `FormBuilder` et `NonNullableFormBuilder` {#formbuilder-and-nonnullableformbuilder}

La classe `FormBuilder` a elle aussi été mise à jour pour prendre en charge les nouveaux types, de la même manière que dans les exemples ci-dessus.

De plus, un builder supplémentaire est disponible : `NonNullableFormBuilder`. Ce type est un raccourci pour spécifier `{nonNullable: true}` sur chaque contrôle, et peut éliminer beaucoup de code répétitif dans les grands formulaires non nullables. Vous pouvez y accéder via la propriété `nonNullable` d'un `FormBuilder` :

```ts
const fb = new FormBuilder();
const login = fb.nonNullable.group({
  email: '',
  password: '',
});
```

Dans l'exemple ci-dessus, les deux contrôles internes seront non nullables (c'est-à-dire que `nonNullable` sera défini).

Vous pouvez aussi l'injecter sous le nom `NonNullableFormBuilder`.
