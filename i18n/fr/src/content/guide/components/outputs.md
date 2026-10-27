<!-- source: e61cfed5dc7adca9bc98874cb602838a4f4db9ad -->
# Événements personnalisés avec les outputs

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Les composants Angular peuvent définir des événements personnalisés en affectant une propriété à la fonction `output` :

```ts {highlight:[5]}
@Component({
  /*...*/
})
export class ExpandablePanel {
  panelClosed = output<void>();
}
```

```angular-html
<expandable-panel (panelClosed)="savePanelState()" />
```

La fonction `output` renvoie un `OutputEmitterRef`. Vous pouvez émettre un événement en appelant la méthode `emit` sur l'`OutputEmitterRef` :

```ts
this.panelClosed.emit();
```

Angular appelle **outputs** les propriétés initialisées avec la fonction `output`. Vous pouvez utiliser les outputs pour déclencher des événements personnalisés, comme les événements natifs du navigateur tels que `click`.

**Les événements personnalisés d'Angular ne remontent pas dans le DOM**.

**Les noms des outputs sont sensibles à la casse.**

Quand vous étendez une classe de composant, **les outputs sont hérités par la classe enfant.**

La fonction `output` a une signification particulière pour le compilateur Angular. **Vous pouvez appeler `output` exclusivement dans les initialiseurs de propriétés des composants et des directives.**

## Émettre des données d'événement {#emitting-event-data}

Vous pouvez transmettre des données d'événement en appelant `emit` :

```ts
// You can emit primitive values.
this.valueChanged.emit(7);

// You can emit custom event objects
this.thumbDropped.emit({
  pointerX: 123,
  pointerY: 456,
});
```

Quand vous définissez un écouteur d'événements dans un template, vous pouvez accéder aux données de l'événement via la variable `$event` :

```angular-html
<custom-slider (valueChanged)="logValue($event)" />
```

Recevez les données de l'événement dans le composant parent :

```ts
@Component({
 /*...*/
})
export class App {
  logValue(value: number) {
    ...
  }
}

```

## Personnaliser le nom des outputs {#customizing-output-names}

La fonction `output` accepte un paramètre qui vous permet de spécifier un nom différent pour l'événement dans un template :

```ts
@Component(/* ... */)
export class CustomSlider {
  changed = output({alias: 'valueChanged'});
}
```

```angular-html
<custom-slider (valueChanged)="saveVolume()" />
```

Cet alias n'a pas d'effet sur l'utilisation de la propriété dans le code TypeScript.

Même si vous devriez en général éviter de donner des alias aux outputs des composants, cette fonctionnalité peut être utile pour renommer des propriétés tout en conservant un alias pour le nom d'origine, ou pour éviter des collisions avec le nom d'événements natifs du DOM.

## S'abonner aux outputs par programmation {#subscribing-to-outputs-programmatically}

Quand vous créez un composant dynamiquement, vous pouvez vous abonner par programmation aux événements des outputs
depuis l'instance du composant. Le type `OutputRef` inclut une méthode `subscribe` :

```ts
const someComponentRef: ComponentRef<SomeComponent> = viewContainerRef.createComponent(/*...*/);

someComponentRef.instance.someEventProperty.subscribe((eventData) => {
  console.log(eventData);
});
```

Angular nettoie automatiquement les abonnements aux événements quand il détruit des composants ayant des abonnés. Vous pouvez aussi vous désabonner manuellement d'un événement. La fonction `subscribe` renvoie un `OutputRefSubscription` doté d'une méthode `unsubscribe` :

```ts
const eventSubscription = someComponent.someEventProperty.subscribe((eventData) => {
  console.log(eventData);
});

// ...

eventSubscription.unsubscribe();
```

## Choisir les noms d'événements {#choosing-event-names}

Évitez de choisir des noms d'outputs qui entrent en collision avec les événements des éléments du DOM comme HTMLElement. Les collisions de noms créent une confusion : on ne sait plus si la propriété liée appartient au composant ou à l'élément du DOM.

N'ajoutez pas de préfixes aux outputs des composants comme vous le feriez pour les sélecteurs de composants. Comme un élément donné ne peut héberger qu'un seul composant, on peut considérer que toutes les propriétés personnalisées appartiennent au composant.

Utilisez toujours des noms d'outputs en [camelCase](https://en.wikipedia.org/wiki/Camel_case). Évitez de préfixer les noms d'outputs par « on ».

## Utiliser les outputs avec RxJS {#using-outputs-with-rxjs}

Consultez [Interopérabilité RxJS avec les outputs des composants et des directives](ecosystem/rxjs-interop/output-interop) pour plus de détails sur l'interopérabilité entre les outputs et RxJS.

## Déclarer des outputs avec le décorateur `@Output` {#declaring-outputs-with-the-output-decorator}

TIP: Même si l'équipe Angular recommande d'utiliser la fonction `output` pour les nouveaux projets, l'API
d'origine `@Output`, basée sur un décorateur, reste entièrement prise en charge.

Vous pouvez aussi définir des événements personnalisés en affectant à une propriété un nouvel `EventEmitter` et en ajoutant le décorateur `@Output` :

```ts
@Component(/* ... */)
export class ExpandablePanel {
  @Output() panelClosed = new EventEmitter<void>();
}
```

Vous pouvez émettre un événement en appelant la méthode `emit` sur l'`EventEmitter`.

### Alias avec le décorateur `@Output` {#aliases-with-the-output-decorator}

Le décorateur `@Output` accepte un paramètre qui vous permet de spécifier un nom différent pour l'événement dans un template :

```ts
@Component(/* ... */)
export class CustomSlider {
  @Output('valueChanged') changed = new EventEmitter<number>();
}
```

```angular-html
<custom-slider (valueChanged)="saveVolume()" />
```

Cet alias n'a pas d'effet sur l'utilisation de la propriété dans le code TypeScript.

## Spécifier les outputs dans le décorateur `@Component` {#specify-outputs-in-the-component-decorator}

En plus du décorateur `@Output`, vous pouvez aussi spécifier les outputs d'un composant avec la propriété `outputs` du décorateur `@Component`. Cela peut être utile quand un composant hérite d'une propriété d'une classe de base :

```ts
// `CustomSlider` inherits the `valueChanged` property from `BaseSlider`.
@Component({
  /*...*/
  outputs: ['valueChanged'],
})
export class CustomSlider extends BaseSlider {}
```

Vous pouvez en plus spécifier un alias d'output dans la liste `outputs` en plaçant l'alias après un deux-points dans la chaîne :

```ts
// `CustomSlider` inherits the `valueChanged` property from `BaseSlider`.
@Component({
  /*...*/
  outputs: ['valueChanged: volumeChanged'],
})
export class CustomSlider extends BaseSlider {}
```
