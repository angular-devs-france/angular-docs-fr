<!-- source: cfa963c6683a3502e9c9d75fadfebb55024acdb1 -->
# État dépendant avec `linkedSignal`

Vous pouvez utiliser la fonction `signal` pour contenir un état dans votre code Angular. Parfois, cet état dépend d'un _autre_ état. Par exemple, imaginez un composant qui permet à l'utilisateur de sélectionner un mode de livraison pour une commande :

```typescript
@Component({
  /* ... */
})
export class ShippingMethodPicker {
  shippingOptions: Signal<ShippingMethod[]> = getShippingOptions();

  // Select the first shipping option by default.
  selectedOption = signal(this.shippingOptions()[0]);

  changeShipping(newOptionIndex: number) {
    this.selectedOption.set(this.shippingOptions()[newOptionIndex]);
  }
}
```

Dans cet exemple, `selectedOption` vaut par défaut la première option, mais change si l'utilisateur sélectionne une autre option. Mais `shippingOptions` est un signal — sa valeur peut changer ! Si `shippingOptions` change, `selectedOption` peut contenir une valeur qui n'est plus une option valide.

**La fonction `linkedSignal` vous permet de créer un signal qui contient un état intrinsèquement _lié_ à un autre état.** Si l'on reprend l'exemple ci-dessus, `linkedSignal` peut remplacer `signal` :

```ts
@Component({
  /* ... */
})
export class ShippingMethodPicker {
  shippingOptions: Signal<ShippingMethod[]> = getShippingOptions();

  // Initialize selectedOption to the first shipping option.
  selectedOption = linkedSignal(() => this.shippingOptions()[0]);

  changeShipping(index: number) {
    this.selectedOption.set(this.shippingOptions()[index]);
  }
}
```

`linkedSignal` fonctionne comme `signal`, à une différence près : au lieu de passer une valeur par défaut, vous passez une _fonction de calcul_, exactement comme pour `computed`. Quand la valeur du calcul change, la valeur du `linkedSignal` devient le résultat du calcul. Cela permet de garantir que le `linkedSignal` a toujours une valeur valide.

L'exemple suivant montre comment la valeur d'un `linkedSignal` peut changer en fonction de l'état auquel il est lié :

```ts
const shippingOptions = signal(['Ground', 'Air', 'Sea']);
const selectedOption = linkedSignal(() => shippingOptions()[0]);
console.log(selectedOption()); // 'Ground'

selectedOption.set(shippingOptions()[2]);
console.log(selectedOption()); // 'Sea'

shippingOptions.set(['Email', 'Will Call', 'Postal service']);
console.log(selectedOption()); // 'Email'
```

## Prendre en compte l'état précédent {#accounting-for-previous-state}

Dans certains cas, le calcul d'un `linkedSignal` doit prendre en compte la valeur précédente du `linkedSignal`.

Dans l'exemple ci-dessus, `selectedOption` revient toujours à la première option quand `shippingOptions` change. Vous pouvez cependant vouloir conserver la sélection de l'utilisateur si l'option qu'il a choisie se trouve toujours dans la liste. Pour cela, vous pouvez créer un `linkedSignal` avec une _source_ et un _calcul_ séparés :

```ts
interface ShippingMethod {
  id: number;
  name: string;
}

@Component({
  /* ... */
})
export class ShippingMethodPicker {
  constructor() {
    this.changeShipping(2);
    this.changeShippingOptions();
    console.log(this.selectedOption()); // {"id":2,"name":"Postal Service"}
  }

  shippingOptions = signal<ShippingMethod[]>([
    {id: 0, name: 'Ground'},
    {id: 1, name: 'Air'},
    {id: 2, name: 'Sea'},
  ]);

  selectedOption = linkedSignal<ShippingMethod[], ShippingMethod>({
    // `selectedOption` is set to the `computation` result whenever this `source` changes.
    source: this.shippingOptions,
    computation: (newOptions, previous) => {
      // If the newOptions contain the previously selected option, preserve that selection.
      // Otherwise, default to the first option.
      return newOptions.find((opt) => opt.id === previous?.value.id) ?? newOptions[0];
    },
  });

  changeShipping(index: number) {
    this.selectedOption.set(this.shippingOptions()[index]);
  }

  changeShippingOptions() {
    this.shippingOptions.set([
      {id: 0, name: 'Email'},
      {id: 1, name: 'Sea'},
      {id: 2, name: 'Postal Service'},
    ]);
  }
}
```

Quand vous créez un `linkedSignal`, vous pouvez passer un objet avec des propriétés `source` et `computation` séparées au lieu de fournir uniquement un calcul.

La `source` peut être n'importe quel signal, comme un `computed` ou un `input` de composant. Le `linkedSignal` met à jour sa valeur quand la `source` change ou quand l'un des signals référencés dans `computation` change, en prenant comme valeur le résultat de la `computation` fournie.

La `computation` est une fonction qui reçoit la nouvelle valeur de `source` et un objet `previous`. L'objet `previous` a deux propriétés : `previous.source` est la valeur précédente de `source`, et `previous.value` est la valeur précédente du `linkedSignal`. Vous pouvez utiliser ces valeurs précédentes pour déterminer le nouveau résultat du calcul.

HELPFUL: Quand vous utilisez le paramètre `previous`, il est nécessaire de fournir explicitement les arguments de type génériques de `linkedSignal`. Le premier type générique correspond au type de `source` et le second détermine le type de sortie de `computation`.

## Comparaison d'égalité personnalisée {#custom-equality-comparison}

Comme tout autre signal, `linkedSignal` peut être configuré avec une fonction d'égalité personnalisée. Cette fonction est utilisée par les dépendances en aval pour déterminer si la valeur du `linkedSignal` (le résultat d'un calcul) a changé :

```typescript
const activeUser = signal({id: 123, name: 'Morgan', isAdmin: true});

const activeUserEditCopy = linkedSignal(() => activeUser(), {
  // Consider the user as the same if it's the same `id`.
  equal: (a, b) => a.id === b.id,
});

// Or, if separating `source` and `computation`
const activeUserEditCopy = linkedSignal({
  source: activeUser,
  computation: (user) => user,
  equal: (a, b) => a.id === b.id,
});
```

## Personnaliser l'opération set {#customizing-the-set-operation}

Vous pouvez parfois vouloir que les opérations `set` et `update` d'un `linkedSignal` écrivent dans la source de vérité au lieu de mettre à jour directement la valeur du `linkedSignal`. Vous pouvez personnaliser ce comportement en passant une fonction `set` dans les options.

La fonction `set` personnalisée reçoit deux arguments :

1. La nouvelle valeur à définir.
2. Une fonction `rawSet`, que vous pouvez appeler pour mettre à jour directement l'état interne du `linkedSignal` (comme le fait le comportement par défaut).

NOTE: Utiliser `rawSet` vous permet de mettre à jour directement la valeur du `linkedSignal`. Cela peut être utile pour éviter l'exécution du calcul, par exemple s'il s'agit d'une dérivation coûteuse et que vous connaissez déjà le résultat.

### Écrire dans un signal source {#writing-back-to-a-source-signal}

Prenons un composant qui affiche une température en Fahrenheit et permet de la modifier, mais qui utilise un signal en Celsius comme source de vérité :

```typescript
const tempC = signal(0);
const tempF = linkedSignal(() => (tempC() * 9) / 5 + 32, {
  set: (valF) => tempC.set(((valF - 32) * 5) / 9),
});

console.log(tempF()); // 32

// Setting Fahrenheit updates Celsius, which reactively updates Fahrenheit
tempF.set(212);
console.log(tempC()); // 100
console.log(tempF()); // 212
```

### Mettre à jour une propriété dans un objet parent {#updating-a-property-inside-a-parent-object}

Un autre scénario courant consiste à mettre à jour une propriété précise dans un objet parent. Le parent est contenu dans un signal, et vous vous liez à une propriété imbriquée :

```typescript
interface Order {
  id: number;
  shippingMethod: string;
}

const order = signal<Order>({
  id: 42,
  shippingMethod: 'Ground',
});

const shippingMethod = linkedSignal(() => order().shippingMethod, {
  set: (newMethod) => {
    // Perform an immutable update to write the change back to the order
    order.update((currentOrder) => ({
      ...currentOrder,
      shippingMethod: newMethod,
    }));
  },
});

console.log(shippingMethod()); // 'Ground'

// Updating the shippingMethod updates the parent order object
shippingMethod.set('Air');
console.log(order()); // { id: 42, shippingMethod: 'Air' }
console.log(shippingMethod()); // 'Air'
```
