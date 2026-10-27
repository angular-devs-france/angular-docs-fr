<!-- source: 68f36ff1abfdac24f2bd2ea025f2d4a3bda88417 -->
# Rendre des composants par programmation

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Lisez-le d'abord si vous découvrez Angular.

En plus d'utiliser un composant directement dans un template, vous pouvez aussi rendre des composants dynamiquement,
par programmation. C'est utile quand un composant n'est pas connu au départ (et ne peut donc pas
être référencé directement dans un template) et qu'il dépend de certaines conditions.

Il existe deux façons principales de rendre un composant par programmation : dans un template avec `NgComponentOutlet`,
ou dans votre code TypeScript avec `ViewContainerRef`.

HELPFUL: pour les cas d'usage de lazy loading (par exemple si vous voulez retarder le chargement d'un composant lourd), envisagez
plutôt d'utiliser la [fonctionnalité `@defer`](/guide/templates/defer) intégrée. La fonctionnalité `@defer` permet d'extraire automatiquement le code
de tous les composants, directives et pipes situés dans le bloc `@defer` dans des chunks JavaScript
séparés, chargés uniquement quand c'est nécessaire, selon les déclencheurs configurés.

## Utiliser NgComponentOutlet {#using-ngcomponentoutlet}

`NgComponentOutlet` est une directive structurelle qui rend dynamiquement un composant donné dans un
template.

```angular-ts
import {NgComponentOutlet} from '@angular/common';
import {Component, input, computed} from '@angular/core';

@Component({/*...*/})
export class AdminBio {
  /* ... */
}

@Component({/*...*/})
export class StandardBio {
  /* ... */
}

@Component({
  imports: [NgComponentOutlet],
  template: `
    <p>Profile for {{ user().name }}</p>
    <ng-container *ngComponentOutlet="bioComponent()" />
  `,
})
export class CustomDialog {
  user = input.required<User>();

  bioComponent = computed(() => {
    return this.user().isAdmin ? AdminBio : StandardBio;
  });
}
```

### Passer des inputs aux composants rendus dynamiquement {#passing-inputs-to-dynamically-rendered-components}

Vous pouvez passer des inputs au composant rendu dynamiquement avec la propriété `ngComponentOutletInputs`. Cette propriété accepte un objet dont les clés sont les noms des inputs et les valeurs, les valeurs des inputs.

```angular-ts
@Component({
  selector: 'user-greeting',
  template: `
    <div>
      <p>User: {{ username() }}</p>
      <p>Role: {{ role() }}</p>
    </div>
  `,
})
export class UserGreeting {
  username = input.required<string>();
  role = input('guest');
}

@Component({
  selector: 'profile-view',
  imports: [NgComponentOutlet],
  template: `<ng-container *ngComponentOutlet="greetingComponent; inputs: greetingInputs()" />`,
})
export class ProfileView {
  greetingComponent = UserGreeting;
  greetingInputs = signal({username: 'ngAwesome', role: 'admin'});
}
```

Les inputs sont mis à jour chaque fois que le signal `greetingInputs` change, ce qui garde le composant dynamique synchronisé avec l'état du parent.

### Fournir une projection de contenu {#providing-content-projection}

Utilisez `ngComponentOutletContent` pour passer du contenu projeté au composant rendu dynamiquement. C'est utile quand le composant dynamique utilise `<ng-content>` pour afficher du contenu.

```angular-ts
@Component({
  selector: 'card-wrapper',
  template: `
    <div class="card">
      <ng-content />
    </div>
  `,
})
export class CardWrapper {}

@Component({
  imports: [NgComponentOutlet],
  template: `
    <ng-container *ngComponentOutlet="cardComponent; content: cardContent()" />

    <ng-template #contentTemplate>
      <h3>Dynamic Content</h3>
      <p>This content is projected into the card.</p>
    </ng-template>
  `,
})
export class DynamicCard {
  private vcr = inject(ViewContainerRef);
  cardComponent = CardWrapper;

  private contentTemplate = viewChild<TemplateRef<unknown>>('contentTemplate');

  cardContent = computed(() => {
    const template = this.contentTemplate();
    if (!template) return [];
    // Returns an array of projection slots. Each element represents one <ng-content> slot.
    // CardWrapper has one <ng-content>, so we return an array with one element.
    return [this.vcr.createEmbeddedView(template).rootNodes];
  });
}
```

NOTE: L'hydratation ne prend pas en charge la projection de nœuds du DOM créés avec les API natives du DOM. Cela provoque une [erreur NG0503](/errors/NG0503). Utilisez les API d'Angular pour créer le contenu projeté ou ajoutez `ngSkipHydration` au composant.

### Fournir des injectors {#providing-injectors}

Vous pouvez fournir un injector personnalisé au composant créé dynamiquement avec `ngComponentOutletInjector`. C'est utile pour fournir des services ou une configuration propres au composant.

```angular-ts
export const THEME_DATA = new InjectionToken<string>('THEME_DATA', {
  factory: () => 'light',
});

@Component({
  selector: 'themed-panel',
  template: `<div [class]="theme">...</div>`,
})
export class ThemedPanel {
  theme = inject(THEME_DATA);
}

@Component({
  selector: 'dynamic-panel',
  imports: [NgComponentOutlet],
  template: `<ng-container *ngComponentOutlet="panelComponent; injector: customInjector" />`,
})
export class DynamicPanel {
  panelComponent = ThemedPanel;

  customInjector = Injector.create({
    providers: [{provide: THEME_DATA, useValue: 'dark'}],
  });
}
```

### Accéder à l'instance du composant {#accessing-the-component-instance}

Vous pouvez accéder à l'instance du composant créé dynamiquement grâce à la fonctionnalité `exportAs` de la directive :

```angular-ts
@Component({
  selector: 'counter',
  template: `<p>Count: {{ count() }}</p>`,
})
export class Counter {
  count = signal(0);
  increment() {
    this.count.update((c) => c + 1);
  }
}

@Component({
  imports: [NgComponentOutlet],
  template: `
    <ng-container [ngComponentOutlet]="counterComponent" #outlet="ngComponentOutlet" />

    <button (click)="outlet.componentInstance?.increment()">Increment</button>
  `,
})
export class CounterHost {
  counterComponent = Counter;
}
```

NOTE: La propriété `componentInstance` vaut `null` avant que le composant soit rendu.

Consultez la [référence de l'API NgComponentOutlet](api/common/NgComponentOutlet) pour en savoir plus sur les
capacités de la directive.

## Utiliser ViewContainerRef {#using-viewcontainerref}

Un **view container** est un nœud de l'arbre des composants d'Angular qui peut contenir du contenu. Tout composant
ou toute directive peut injecter `ViewContainerRef` pour obtenir une référence au view container correspondant à
l'emplacement de ce composant ou de cette directive dans le DOM.

Vous pouvez utiliser la méthode `createComponent`de `ViewContainerRef` pour créer et rendre dynamiquement un
composant. Quand vous créez un nouveau composant avec un `ViewContainerRef`, Angular l'ajoute dans le
DOM comme frère suivant du composant ou de la directive qui a injecté le `ViewContainerRef`.

```angular-ts
@Component({
  selector: 'leaf-content',
  template: `This is the leaf content`,
})
export class LeafContent {}

@Component({
  selector: 'outer-container',
  template: `
    <p>This is the start of the outer container</p>
    <inner-item />
    <p>This is the end of the outer container</p>
  `,
})
export class OuterContainer {}

@Component({
  selector: 'inner-item',
  template: `<button (click)="loadContent()">Load content</button>`,
})
export class InnerItem {
  private viewContainer = inject(ViewContainerRef);

  loadContent() {
    this.viewContainer.createComponent(LeafContent);
  }
}
```

Dans l'exemple ci-dessus, un clic sur le bouton « Load content » produit la structure de DOM suivante

```angular-html
<outer-container>
  <p>This is the start of the outer container</p>
  <inner-item>
    <button>Load content</button>
  </inner-item>
  <leaf-content>This is the leaf content</leaf-content>
  <p>This is the end of the outer container</p>
</outer-container>
```

## Lazy loading de composants {#lazy-loading-components}

HELPFUL: si vous voulez charger certains composants en lazy loading, vous pouvez envisager d'utiliser plutôt la [fonctionnalité `@defer`](/guide/templates/defer)
intégrée.

Si votre cas d'usage n'est pas couvert par la fonctionnalité `@defer`, vous pouvez utiliser `NgComponentOutlet` ou
`ViewContainerRef` avec un [import dynamique](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/import) JavaScript standard.

```angular-ts
@Component({
  ...,
  template: `
    <section>
      <h2>Basic settings</h2>
      <basic-settings />
    </section>
    <section>
      <h2>Advanced settings</h2>
      @if(!advancedSettings) {
        <button (click)="loadAdvanced()">
          Load advanced settings
        </button>
      }
      <ng-container *ngComponentOutlet="advancedSettings" />
    </section>
  `
})
export class AdminSettings {
  advancedSettings: {new(): AdvancedSettings} | undefined;

  async loadAdvanced() {
    const { AdvancedSettings } = await import('path/to/advanced_settings.js');
    this.advancedSettings = AdvancedSettings;
  }
}
```

L'exemple ci-dessus charge et affiche `AdvancedSettings` lors d'un clic sur un bouton.

## Lier des inputs et des outputs et définir des host directives à la création {#binding-inputs-outputs-and-setting-host-directives-at-creation}

Lors de la création dynamique de composants, définir manuellement les inputs et s'abonner aux outputs peut être source d'erreurs. Vous devez souvent écrire du code supplémentaire rien que pour mettre en place les bindings une fois le composant instancié.

Pour simplifier cela, `createComponent` et `ViewContainerRef.createComponent` acceptent tous deux un tableau `bindings` avec des helpers comme `inputBinding()`, `outputBinding()` et `twoWayBinding()` pour configurer les inputs et les outputs dès le départ. Vous pouvez aussi spécifier un tableau `directives` pour appliquer des host directives. Cela permet de créer des composants par programmation avec des bindings semblables à ceux d'un template, en un seul appel déclaratif.

### Host view avec `ViewContainerRef.createComponent` {#host-view-using-viewcontainerrefcreatecomponent}

`ViewContainerRef.createComponent` crée un composant et insère automatiquement sa host view et son host element dans la hiérarchie de vues du conteneur, à l'emplacement du conteneur. Utilisez-le quand le composant dynamique doit faire partie de la structure logique et visuelle du conteneur (par exemple, pour ajouter des éléments de liste ou une UI en ligne).

À l'inverse, l'API standalone `createComponent` n'attache le nouveau composant à aucune vue ni à aucun emplacement du DOM existant : elle renvoie un `ComponentRef` et vous laisse contrôler explicitement où placer le host element du composant.

```angular-ts
import {Component, input, model, output} from '@angular/core';

@Component({
  selector: 'app-warning',
  template: `
    @if (isExpanded()) {
      <section>
        <p>Warning: Action needed!</p>
        <button (click)="close.emit(true)">Close</button>
      </section>
    }
  `,
})
export class AppWarning {
  readonly canClose = input.required<boolean>();
  readonly isExpanded = model<boolean>();
  readonly close = output<boolean>();
}
```

```ts
import {
  Component,
  ViewContainerRef,
  signal,
  inputBinding,
  outputBinding,
  twoWayBinding,
  inject,
} from '@angular/core';
import {FocusTrap} from '@angular/cdk/a11y';
import {ThemeDirective} from '../theme.directive';

@Component({
  template: `<ng-container #container />`,
})
export class Host {
  private vcr = inject(ViewContainerRef);
  readonly canClose = signal(true);
  readonly isExpanded = signal(true);

  showWarning() {
    const compRef = this.vcr.createComponent(AppWarning, {
      bindings: [
        inputBinding('canClose', this.canClose),
        twoWayBinding('isExpanded', this.isExpanded),
        outputBinding<boolean>('close', (confirmed) => {
          console.log('Closed with result:', confirmed);
        }),
      ],
      directives: [
        FocusTrap,
        {type: ThemeDirective, bindings: [inputBinding('theme', () => 'warning')]},
      ],
    });
  }
}
```

Dans l'exemple ci-dessus, le composant dynamique **AppWarning** est créé avec son input `canClose` lié à un signal réactif, un two-way binding sur son état `isExpanded` et un écouteur d'output pour `close`. `FocusTrap` et `ThemeDirective` sont attachées au host element via `directives`.

### Popup attachée à `document.body` avec `createComponent` + `hostElement` {#popup-attached-to-documentbody-with-createcomponent--hostelement}

Utilisez cette approche pour un rendu en dehors de la hiérarchie de vues actuelle (par exemple, des overlays). Le `hostElement` fourni devient le host du composant dans le DOM, si bien qu'Angular ne crée pas de nouvel élément correspondant au selector. Vous pouvez configurer directement les **bindings**.

```ts
import {
  ApplicationRef,
  createComponent,
  EnvironmentInjector,
  inject,
  Injectable,
  inputBinding,
  outputBinding,
  Service,
} from '@angular/core';
import {Popup} from './popup';

@Service()
export class PopupService {
  private readonly injector = inject(EnvironmentInjector);
  private readonly appRef = inject(ApplicationRef);

  show(message: string) {
    // Create a host element for the popup
    const host = document.createElement('popup-host');

    // Create the component and bind in one call
    const ref = createComponent(Popup, {
      environmentInjector: this.injector,
      hostElement: host,
      bindings: [
        inputBinding('message', () => message),
        outputBinding('closed', () => {
          document.body.removeChild(host);
          this.appRef.detachView(ref.hostView);
          ref.destroy();
        }),
      ],
    });

    // Registers the component’s view so it participates in change detection cycle.
    this.appRef.attachView(ref.hostView);
    // Inserts the provided host element into the DOM (outside the normal Angular view hierarchy).
    // This is what makes the popup visible on screen, typically used for overlays or modals.
    document.body.appendChild(host);
  }
}
```

## Gérer les erreurs de rendu {#handling-rendering-errors}

IMPORTANT: L'option `onError` et son paramètre `ErrorDetails` sont en [developer preview](reference/releases#developer-preview).

Quand vous créez des composants dynamiquement avec `ViewContainerRef.createComponent` ou la fonction standalone `createComponent`, vous pouvez fournir un callback `onError` dans l'objet d'options pour gérer les erreurs qui surviennent pendant les phases de rendu ou de détection de changements. C'est l'équivalent, par programmation, d'un bloc `@error` dans les templates.

```ts
viewContainerRef.createComponent(DynamicComponent, {
  onError: (err: Error, details: ErrorDetails) => {
    console.error('Component rendering failed:', err);
    // Render an alternative UI or log metrics
  },
});
```

NOTE: Le callback `onError` n'intercepte que les erreurs qui surviennent pendant les phases de rendu ou de détection de changements. Il n'intercepte pas les erreurs qui surviennent pendant l'instanciation du composant (par exemple, dans le constructeur). Angular lève les erreurs de construction de manière synchrone quand vous appelez l'API.
