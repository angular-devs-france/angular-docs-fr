<!-- source: 67ab7ebd06db8df3109217f4152399fc45abe4a6 -->
# Accepter des données avec les propriétés d'input

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular. Si vous connaissez d'autres frameworks web, les propriétés d'input sont similaires aux _props_.

Quand vous utilisez un composant, vous souhaitez généralement lui transmettre des données. Un composant spécifie les données qu'il accepte en déclarant
des **inputs** :

```ts {highlight:[6]}
import {Component, input} from '@angular/core';

@Component(/* ... */)
export class CustomSlider {
  // Declare an input named 'value' with a default value of zero.
  value = input(0);
}
```

Cela vous permet de lier la propriété dans un template :

```angular-html
<custom-slider [value]="50" />
```

Si un input a une valeur par défaut, TypeScript déduit le type à partir de cette valeur par défaut :

```ts
@Component(/* ... */)
export class CustomSlider {
  // TypeScript infers that this input is a number, returning InputSignal<number>.
  value = input(0);
}
```

Vous pouvez déclarer explicitement un type pour l'input en spécifiant un paramètre générique à la fonction.

Si un input sans valeur par défaut n'est pas défini, sa valeur est `undefined` :

```ts
@Component(/* ... */)
export class CustomSlider {
  // Produces an InputSignal<number | undefined> because `value` may not be set.
  value = input<number>();
}
```

**Angular enregistre les inputs de manière statique, à la compilation**. Les inputs ne peuvent pas être ajoutés ni supprimés à l'exécution.

La fonction `input` a une signification particulière pour le compilateur Angular. **Vous pouvez appeler `input` exclusivement dans les initialiseurs de propriétés des composants et des directives.**

Quand vous étendez une classe de composant, **les inputs sont hérités par la classe enfant.**

**Les noms des inputs sont sensibles à la casse.**

## Lire les inputs {#reading-inputs}

La fonction `input` renvoie un `InputSignal`. Vous pouvez lire la valeur en appelant le signal :

```ts {highlight:[9]}
import {Component, input, computed} from '@angular/core';

@Component(/* ... */)
export class CustomSlider {
  // Declare an input named 'value' with a default value of zero.
  value = input(0);

  // Create a computed expression that reads the value input
  label = computed(() => `The slider's value is ${this.value()}`);
}
```

Les signals créés par la fonction `input` sont en lecture seule.

## Inputs obligatoires {#required-inputs}

Vous pouvez déclarer qu'un input est obligatoire (`required`) en appelant `input.required` au lieu de `input` :

```ts {highlight:[4]}
@Component(/* ... */)
export class CustomSlider {
  // Declare a required input named value. Returns an `InputSignal<number>`.
  value = input.required<number>();
}
```

Angular impose que les inputs obligatoires _doivent_ être définis quand le composant est utilisé dans un template. Si vous essayez d'utiliser un composant sans spécifier tous ses inputs obligatoires, Angular signale une erreur au moment du build.

Les inputs obligatoires n'incluent pas automatiquement `undefined` dans le paramètre générique de l'`InputSignal` renvoyé.

## Configurer les inputs {#configuring-inputs}

La fonction `input` accepte en second paramètre un objet de configuration qui vous permet de modifier le fonctionnement de l'input.

### Transformations d'input {#input-transforms}

Vous pouvez spécifier une fonction `transform` pour modifier la valeur d'un input quand elle est définie par Angular.

```ts {highlight:[6]}
@Component({
  selector: 'custom-slider',
  /*...*/
})
export class CustomSlider {
  label = input('', {transform: trimString});
}

function trimString(value: string | undefined): string {
  return value?.trim() ?? '';
}
```

```angular-html
<custom-slider [label]="systemVolume" />
```

Dans l'exemple ci-dessus, chaque fois que la valeur de `systemVolume` change, Angular exécute `trimString` et affecte le résultat à `label`.

Le cas d'usage le plus courant des transformations d'input est d'accepter un éventail plus large de types de valeurs dans les templates, incluant souvent `null` et `undefined`.

**Une fonction de transformation d'input doit pouvoir être analysée statiquement au moment du build.** Vous ne pouvez pas définir des fonctions de transformation de façon conditionnelle ni comme résultat de l'évaluation d'une expression.

**Les fonctions de transformation d'input devraient toujours être des [fonctions pures](https://en.wikipedia.org/wiki/Pure_function).** Dépendre d'un état extérieur à la fonction de transformation peut entraîner un comportement imprévisible.

#### Vérification de type {#type-checking}

Quand vous spécifiez une transformation d'input, le type du paramètre de la fonction de transformation détermine les types de valeurs qui peuvent être affectées à l'input dans un template.

```ts
@Component(/* ... */)
export class CustomSlider {
  widthPx = input('', {transform: appendPx});
}

function appendPx(value: number): string {
  return `${value}px`;
}
```

Dans l'exemple ci-dessus, l'input `widthPx` accepte un `number` alors que la propriété `InputSignal` renvoie une `string`.

#### Transformations intégrées {#built-in-transformations}

Angular inclut deux fonctions de transformation intégrées pour les deux scénarios les plus courants : la conversion de valeurs en booléens et en nombres.

```ts
import {Component, input, booleanAttribute, numberAttribute} from '@angular/core';

@Component(/* ... */)
export class CustomSlider {
  disabled = input(false, {transform: booleanAttribute});
  value = input(0, {transform: numberAttribute});
}
```

`booleanAttribute` imite le comportement des [attributs booléens](https://developer.mozilla.org/docs/Glossary/Boolean/HTML) HTML standard, où la
_présence_ de l'attribut indique une valeur « true ». Cependant, le `booleanAttribute` d'Angular traite la chaîne littérale `"false"` comme le booléen `false`.

`numberAttribute` tente de convertir la valeur donnée en nombre, et produit `NaN` si la conversion échoue.

### Alias d'input {#input-aliases}

Vous pouvez spécifier l'option `alias` pour changer le nom d'un input dans les templates.

```ts {highlight:[3]}
@Component(/* ... */)
export class CustomSlider {
  value = input(0, {alias: 'sliderValue'});
}
```

```angular-html
<custom-slider [sliderValue]="50" />
```

Cet alias n'a pas d'effet sur l'utilisation de la propriété dans le code TypeScript.

Même si vous devriez en général éviter de donner des alias aux inputs des composants, cette fonctionnalité peut être utile pour renommer des propriétés tout en conservant un alias pour le nom d'origine, ou pour éviter des collisions avec le nom de propriétés natives des éléments du DOM.

## Inputs model {#model-inputs}

Les **inputs model** sont un type particulier d'input qui permet à un composant de propager de nouvelles valeurs vers son composant parent.

Quand vous créez un composant, vous pouvez définir un input model de la même manière qu'un input standard.

Les deux types d'input permettent de lier une valeur à la propriété. Cependant, **les inputs model permettent à l'auteur du composant d'écrire des valeurs dans la propriété**. Si la propriété est liée par un two-way binding, la nouvelle valeur se propage à ce binding.

```ts
@Component(/* ... */)
export class CustomSlider {
  // Define a model input named "value".
  value = model(0);

  increment() {
    // Update the model input with a new value, propagating the value to any bindings.
    this.value.update((oldValue) => oldValue + 10);
  }
}

@Component({
  /* ... */
  // Using the two-way binding syntax means that any changes to the slider's
  // value automatically propagate back to the `volume` signal.
  // Note that this binding uses the signal *instance*, not the signal value.
  template: `<custom-slider [(value)]="volume" />`,
})
export class MediaControls {
  // Create a writable signal for the `volume` local state.
  volume = signal(0);
}
```

Dans l'exemple ci-dessus, `CustomSlider` peut écrire des valeurs dans son input model `value`, qui propage ensuite ces valeurs vers le signal `volume` de `MediaControls`. Ce binding garde les valeurs de `value` et `volume` synchronisées. Notez que le binding transmet l'instance du signal `volume`, et non la _valeur_ du signal.

Pour le reste, les inputs model fonctionnent comme les inputs standard. Vous pouvez lire la valeur en appelant la fonction du signal, y compris dans des [contextes réactifs](guide/signals#reactive-contexts) comme `computed` et `effect`.

Consultez [Two-way binding](guide/templates/two-way-binding) pour plus de détails sur le two-way binding dans les templates.

### Two-way binding avec des propriétés simples {#two-way-binding-with-plain-properties}

Vous pouvez lier une propriété JavaScript simple à un input model.

```angular-ts
@Component({
  /* ... */
  // `value` is a model input.
  // The parenthesis-inside-square-brackets syntax (aka "banana-in-a-box") creates a two-way binding
  template: '<custom-slider [(value)]="volume" />',
})
export class MediaControls {
  protected volume = 0;
}
```

Dans l'exemple ci-dessus, `CustomSlider` peut écrire des valeurs dans son input model `value`, qui propage ensuite ces valeurs vers la propriété `volume` de `MediaControls`. Ce binding garde les valeurs de `value` et `volume` synchronisées.

### Événements `change` implicites {#implicit-change-events}

Quand vous déclarez un input model dans un composant ou une directive, Angular crée automatiquement un [output](guide/components/outputs) correspondant pour ce model. Le nom de l'output est le nom de l'input model suffixé par « Change ».

```ts
@Directive(/* ... */)
export class CustomCheckbox {
  // This automatically creates an output named "checkedChange".
  // Can be subscribed to using `(checkedChange)="handler()"` in the template.
  checked = model(false);
}
```

Angular émet cet événement de changement chaque fois que vous écrivez une nouvelle valeur dans l'input model en appelant ses méthodes `set` ou `update`.

Consultez [Événements personnalisés avec les outputs](guide/components/outputs) pour plus de détails sur les outputs.

### Personnaliser les inputs model {#customizing-model-inputs}

Vous pouvez marquer un input model comme [obligatoire](#required-inputs) ou lui fournir un [alias](#input-aliases) de la même manière qu'un input standard.

Les inputs model ne prennent pas en charge les transformations d'input.

### Quand utiliser les inputs model {#when-to-use-model-inputs}

Utilisez les inputs model quand vous voulez qu'un composant prenne en charge le two-way binding. C'est généralement approprié quand un composant sert à modifier une valeur en fonction des interactions de l'utilisateur. Le plus souvent, les contrôles de formulaire personnalisés, comme un sélecteur de date ou une combobox, devraient utiliser des inputs model pour leur valeur principale.

## Choisir les noms des inputs {#choosing-input-names}

Évitez de choisir des noms d'inputs qui entrent en collision avec les propriétés des éléments du DOM comme HTMLElement. Les collisions de noms créent une confusion : on ne sait plus si la propriété liée appartient au composant ou à l'élément du DOM.

N'ajoutez pas de préfixes aux inputs des composants comme vous le feriez pour les selectors de composants. Comme un élément donné ne peut héberger qu'un seul composant, on peut considérer que toutes les propriétés personnalisées appartiennent au composant.

## Déclarer des inputs avec le décorateur `@Input` {#declaring-inputs-with-the-input-decorator}

TIP: Même si l'équipe Angular recommande d'utiliser la fonction `input` basée sur les signals pour les nouveaux projets, l'API d'origine `@Input`, basée sur un décorateur, reste entièrement prise en charge.

Vous pouvez aussi déclarer les inputs d'un composant en ajoutant le décorateur `@Input` à une propriété :

```ts {highlight:[3]}
@Component(/* ... */)
export class CustomSlider {
  @Input() value = 0;
}
```

Le binding à un input est identique pour les inputs basés sur les signals et pour ceux basés sur un décorateur :

```angular-html
<custom-slider [value]="50" />
```

### Personnaliser les inputs basés sur un décorateur {#customizing-decorator-based-inputs}

Le décorateur `@Input` accepte un objet de configuration qui vous permet de modifier le fonctionnement de l'input.

#### Inputs obligatoires {#required-inputs-decorator}

Vous pouvez spécifier l'option `required` pour imposer qu'un input donné ait toujours une valeur.

```ts {highlight:[3]}
@Component(/* ... */)
export class CustomSlider {
  @Input({required: true}) value = 0;
}
```

Si vous essayez d'utiliser un composant sans spécifier tous ses inputs obligatoires, Angular signale une erreur au moment du build.

#### Transformations d'input {#input-transforms-decorator}

Vous pouvez spécifier une fonction `transform` pour modifier la valeur d'un input quand elle est définie par Angular. Cette fonction de transformation fonctionne exactement comme les fonctions de transformation des inputs basés sur les signals décrites ci-dessus.

```ts {highlight:[6]}
@Component({
  selector: 'custom-slider',
  ...
})
export class CustomSlider {
  @Input({transform: trimString}) label = '';
}

function trimString(value: string | undefined) {
  return value?.trim() ?? '';
}
```

#### Alias d'input {#input-aliases-decorator}

Vous pouvez spécifier l'option `alias` pour changer le nom d'un input dans les templates.

```ts {highlight:[3]}
@Component(/* ... */)
export class CustomSlider {
  @Input({alias: 'sliderValue'}) value = 0;
}
```

```angular-html
<custom-slider [sliderValue]="50" />
```

Le décorateur `@Input` accepte aussi l'alias comme premier paramètre, à la place de l'objet de configuration.

Les alias d'input fonctionnent de la même manière que pour les inputs basés sur les signals décrits ci-dessus.

### Inputs avec getters et setters {#inputs-with-getters-and-setters}

Avec les inputs basés sur un décorateur, une propriété implémentée avec un getter et un setter peut être un input :

```ts
export class CustomSlider {
  @Input()
  get value(): number {
    return this.internalValue;
  }

  set value(newValue: number) {
    this.internalValue = newValue;
  }

  private internalValue = 0;
}
```

Vous pouvez même créer un input _en écriture seule_ en ne définissant qu'un setter public :

```ts
export class CustomSlider {
  @Input()
  set value(newValue: number) {
    this.internalValue = newValue;
  }

  private internalValue = 0;
}
```

**Préférez les transformations d'input aux getters et setters** lorsque c'est possible.

Évitez les getters et setters complexes ou coûteux. Angular peut invoquer le setter d'un input plusieurs fois, ce qui peut nuire aux performances de l'application si le setter effectue des opérations coûteuses, comme des manipulations du DOM.

## Spécifier les inputs dans le décorateur `@Component` {#specify-inputs-in-the-component-decorator}

En plus du décorateur `@Input`, vous pouvez aussi spécifier les inputs d'un composant avec la propriété `inputs` du décorateur `@Component`. Cela peut être utile quand un composant hérite d'une propriété d'une classe de base :

```ts {highlight:[4]}
// `CustomSlider` inherits the `disabled` property from `BaseSlider`.
@Component({
  ...,
  inputs: ['disabled'],
})
export class CustomSlider extends BaseSlider { }
```

Vous pouvez en plus spécifier un alias d'input dans la liste `inputs` en plaçant l'alias après un deux-points dans la chaîne :

```ts {highlight:[4]}
// `CustomSlider` inherits the `disabled` property from `BaseSlider`.
@Component({
  ...,
  inputs: ['disabled: sliderDisabled'],
})
export class CustomSlider extends BaseSlider { }
```
