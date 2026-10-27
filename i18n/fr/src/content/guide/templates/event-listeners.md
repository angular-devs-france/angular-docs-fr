<!-- source: 4871cf446a746fc1d5ede66b7bd5f48e0b005015 -->
# Ajouter des écouteurs d'événements

Angular permet de définir des écouteurs d'événements sur un élément de votre template en indiquant le nom de l'événement entre parenthèses, accompagné d'une instruction qui s'exécute chaque fois que l'événement se produit.

## Écouter les événements natifs {#listening-to-native-events}

Pour ajouter des écouteurs d'événements à un élément HTML, entourez l'événement de parenthèses, `()`, ce qui vous permet de spécifier une instruction d'écoute.

```angular-ts
@Component({
  template: `
    <input type="text" (keyup)="updateField()" />
  `,
  ...
})
export class App{
  updateField(): void {
    console.log('Field is updated!');
  }
}
```

Dans cet exemple, Angular appelle `updateField` chaque fois que l'élément `<input>` émet un événement `keyup`.

Vous pouvez ajouter des écouteurs pour n'importe quel événement natif, comme `click`, `keydown`, `mouseover`, etc. Pour en savoir plus, consultez [la liste de tous les événements disponibles sur les éléments sur MDN](https://developer.mozilla.org/en-US/docs/Web/API/Element#events).

## Accéder à l'argument de l'événement {#accessing-the-event-argument}

Dans chaque écouteur d'événements de template, Angular fournit une variable nommée `$event` qui contient une référence à l'objet de l'événement.

```angular-ts
@Component({
  template: `
    <input type="text" (keyup)="updateField($event)" />
  `,
  ...
})
export class App {
  updateField(event: KeyboardEvent): void {
    console.log(`The user pressed: ${event.key}`);
  }
}
```

## Utiliser les modificateurs de touches {#using-key-modifiers}

Quand vous souhaitez capturer des événements clavier pour une touche précise, vous pourriez écrire du code comme celui-ci :

```angular-ts
@Component({
  template: `
    <input type="text" (keyup)="updateField($event)" />
  `,
  ...
})
export class App {
  updateField(event: KeyboardEvent): void {
    if (event.key === 'Enter') {
      console.log('The user pressed enter in the text field.');
    }
  }
}
```

Cependant, comme ce scénario est courant, Angular vous permet de filtrer les événements en spécifiant une touche précise à l'aide du caractère point (`.`). Le code peut ainsi être simplifié comme suit :

```angular-ts
@Component({
  template: `
    <input type="text" (keyup.enter)="updateField($event)" />
  `,
  ...
})
export class App{
  updateField(event: KeyboardEvent): void {
    console.log('The user pressed enter in the text field.');
  }
}
```

Vous pouvez également ajouter des modificateurs de touches supplémentaires :

```angular-html
<!-- Matches shift and enter -->
<input type="text" (keyup.shift.enter)="updateField($event)" />
```

Angular prend en charge les modificateurs `alt`, `control`, `meta` et `shift`.

Vous pouvez spécifier la touche (key) ou le code que vous souhaitez lier aux événements clavier. Les champs key et code font partie nativement de l'objet d'événement clavier du navigateur. Par défaut, la liaison d'événement suppose que vous souhaitez utiliser les [valeurs Key des événements clavier](https://developer.mozilla.org/docs/Web/API/UI_Events/Keyboard_event_key_values).

Angular vous permet aussi de spécifier les [valeurs Code des événements clavier](https://developer.mozilla.org/docs/Web/API/UI_Events/Keyboard_event_code_values) grâce au suffixe intégré `code`.

```angular-html
<!-- Matches alt and left shift -->
<input type="text" (keydown.code.alt.shift.shiftleft)="updateField($event)" />
```

Cela peut être utile pour gérer les événements clavier de manière cohérente entre différents systèmes d'exploitation. Par exemple, quand vous utilisez la touche Alt sur les appareils macOS, la propriété `key` indique la touche en fonction du caractère déjà modifié par la touche Alt. Une combinaison comme Alt + S renvoie donc une valeur `key` de `'ß'`. La propriété `code`, en revanche, correspond au bouton physique ou virtuel pressé, et non au caractère produit.

## Écouter sur des cibles globales {#listening-on-global-targets}

Les noms de cibles globales peuvent servir de préfixe à un événement. Les 3 cibles globales prises en charge sont `window`, `document` et `body`.

```angular-ts
@Component({
  /* ... */
  host: {
    '(window:click)': 'onWindowClick()',
    '(document:click)': 'onDocumentClick()',
    '(body:click)': 'onBodyClick()',
  },
})
export class MyView {}
```

## Empêcher le comportement par défaut d'un événement {#preventing-event-default-behavior}

Si votre gestionnaire d'événement doit remplacer le comportement natif du navigateur, vous pouvez utiliser la [méthode `preventDefault`](https://developer.mozilla.org/en-US/docs/Web/API/Event/preventDefault) de l'objet de l'événement :

```angular-ts
@Component({
  template: `
    <a href="#overlay" (click)="showOverlay($event)">
  `,
  ...
})
export class App{
  showOverlay(event: PointerEvent): void {
    event.preventDefault();
    console.log('Show overlay without updating the URL!');
  }
}
```

Si l'instruction du gestionnaire d'événement s'évalue à `false`, Angular appelle automatiquement `preventDefault()`, comme pour les [attributs natifs de gestionnaires d'événements](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Attributes#event_handler_attributes). _Privilégiez toujours un appel explicite à `preventDefault`_, car cette approche rend l'intention du code évidente.

## Étendre la gestion des événements {#extend-event-handling}

Le système d'événements d'Angular est extensible grâce à des plugins d'événements personnalisés, enregistrés avec le jeton d'injection `EVENT_MANAGER_PLUGINS`.

### Implémenter un plugin d'événements {#implementing-event-plugin}

Pour créer un plugin d'événements personnalisé, étendez la classe `EventManagerPlugin` et implémentez les méthodes requises.

```ts
import {Injectable} from '@angular/core';
import {EventManagerPlugin} from '@angular/platform-browser';

@Injectable()
export class DebounceEventPlugin extends EventManagerPlugin {
  constructor() {
    super(document);
  }

  // Define which events this plugin supports
  override supports(eventName: string) {
    return /debounce/.test(eventName);
  }

  // Handle the event registration
  override addEventListener(element: HTMLElement, eventName: string, handler: Function) {
    // Parse the event: e.g., "click.debounce.500"
    // event: "click", delay: 500
    const [event, , delay = '300'] = eventName.split('.');

    let timeoutId: ReturnType<typeof setTimeout>;

    const listener = (e: Event) => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        handler(e);
      }, Number(delay));
    };

    element.addEventListener(event, listener);

    // Return cleanup function
    return () => {
      clearTimeout(timeoutId);
      element.removeEventListener(event, listener);
    };
  }
}
```

Enregistrez votre plugin personnalisé avec le jeton `EVENT_MANAGER_PLUGINS` dans les providers de votre application :

```ts
import {bootstrapApplication, EVENT_MANAGER_PLUGINS} from '@angular/platform-browser';
import {App} from './app';
import {DebounceEventPlugin} from './debounce-event-plugin';

bootstrapApplication(App, {
  providers: [
    {
      provide: EVENT_MANAGER_PLUGINS,
      useClass: DebounceEventPlugin,
      multi: true,
    },
  ],
});
```

Une fois le plugin enregistré, vous pouvez utiliser votre syntaxe d'événement personnalisée dans les templates, ainsi qu'avec la propriété `host` :

```angular-ts
@Component({
  template: `
    <input
      type="text"
      (input.debounce.500)="onSearch($event.target.value)"
      placeholder="Search..."
    />
  `,
  ...
})
export class Search {
 onSearch(query: string): void {
    console.log('Searching for:', query);
  }
}
```

```ts
@Component({
  ...,
  host: {
    '(click.debounce.500)': 'handleDebouncedClick()',
  },
})
export class AwesomeCard {
  handleDebouncedClick(): void {
   console.log('Debounced click!');
  }
}
```
