<!-- source: 5cfb2455a00a680cc9c7f57d21134fc61eb178d2 -->
# Créer des harnais pour vos composants

## Avant de commencer {#before-you-start}

TIP: Ce guide suppose que vous avez déjà lu la [vue d'ensemble des harnais de composant](guide/testing/component-harnesses-overview). Lisez-la d'abord si vous découvrez les harnais de composant.

### Quand est-il pertinent de créer un harnais de test ? {#when-does-creating-a-test-harness-make-sense}

L'équipe Angular recommande de créer des harnais de test de composant pour les composants partagés utilisés à de nombreux endroits et qui offrent une certaine interactivité. Cela concerne le plus souvent les libraries de widgets et les composants réutilisables similaires. Les harnais sont précieux dans ces cas, car ils fournissent aux consommateurs de ces composants partagés une API bien prise en charge pour interagir avec un composant. Les tests qui utilisent des harnais peuvent éviter de dépendre de détails d'implémentation peu fiables de ces composants partagés, comme la structure DOM et des event listeners spécifiques.

Pour les composants qui n'apparaissent qu'à un seul endroit, comme une page d'une application, les harnais apportent moins d'avantages. Dans ces situations, les tests d'un composant peuvent raisonnablement dépendre des détails d'implémentation de ce composant, puisque les tests et les composants sont mis à jour en même temps. Les harnais restent toutefois utiles si vous comptez utiliser le harnais à la fois dans des tests unitaires et de bout en bout.

### Installation du CDK {#cdk-installation}

Le [Component Dev Kit (CDK)](https://material.angular.dev/cdk/categories) est un ensemble de primitives de comportement pour construire des composants. Pour utiliser les harnais de composant, installez d'abord `@angular/cdk` depuis npm. Vous pouvez le faire depuis votre terminal avec l'Angular CLI :

```shell
ng add @angular/cdk
```

## Étendre `ComponentHarness` {#extending-componentharness}

La classe abstraite `ComponentHarness` est la classe de base de tous les harnais de composant. Pour créer un harnais de composant personnalisé, étendez `ComponentHarness` et implémentez la propriété statique `hostSelector`.

La propriété `hostSelector` identifie les éléments du DOM qui correspondent à cette sous-classe de harnais. Dans la plupart des cas, le `hostSelector` doit être identique au selector du `Component` ou de la `Directive` correspondant. Par exemple, considérez un simple composant popup :

```ts
@Component({
  selector: 'my-popup',
  template: `
    <button (click)="toggle()">{{ triggerText() }}</button>
    @if (isOpen()) {
      <div class="my-popup-content"><ng-content /></div>
    }
  `,
})
class MyPopup {
  triggerText = input('');

  isOpen = signal(false);

  toggle() {
    this.isOpen.update((value) => !value);
  }
}
```

Dans ce cas, un harnais minimal pour le composant ressemblerait à ceci :

```ts
class MyPopupHarness extends ComponentHarness {
  static hostSelector = 'my-popup';
}
```

Bien que les sous-classes de `ComponentHarness` n'exigent que la propriété `hostSelector`, la plupart des harnais doivent aussi implémenter une méthode statique `with` pour générer des instances de `HarnessPredicate`. La [section sur le filtrage des harnais](guide/testing/using-component-harnesses#filtering-harnesses) aborde ce point plus en détail.

## Trouver des éléments dans le DOM du composant {#finding-elements-in-the-components-dom}

Chaque instance d'une sous-classe de `ComponentHarness` représente une instance particulière du composant correspondant. Vous pouvez accéder au host element du composant via la méthode `host() ` de la classe de base `ComponentHarness`.

`ComponentHarness` propose aussi plusieurs méthodes pour localiser des éléments dans le DOM du composant : `locatorFor()`, `locatorForOptional()` et `locatorForAll()`. Ces méthodes créent des fonctions qui trouvent des éléments, elles ne trouvent pas directement les éléments. Cette approche évite de mettre en cache des références à des éléments obsolètes. Par exemple, quand un bloc `@if` masque puis affiche un élément, le résultat est un nouvel élément DOM ; utiliser des fonctions garantit que les tests référencent toujours l'état actuel du DOM.

Consultez la [page de référence de l'API ComponentHarness](/api/cdk/testing/ComponentHarness) pour la liste détaillée complète des différentes méthodes `locatorFor`.

Par exemple, le `MyPopupHarness` présenté plus haut pourrait fournir des méthodes pour récupérer les éléments déclencheur et contenu comme suit :

```ts
class MyPopupHarness extends ComponentHarness {
  static hostSelector = 'my-popup';

  // Gets the trigger element
  getTriggerElement = this.locatorFor('button');

  // Gets the content element.
  getContentElement = this.locatorForOptional('.my-popup-content');
}
```

## Travailler avec des instances de `TestElement` {#working-with-testelement-instances}

`TestElement` est une abstraction conçue pour fonctionner dans différents environnements de test (tests unitaires, WebDriver, etc.). Quand vous utilisez des harnais, effectuez toutes les interactions avec le DOM via cette interface. Les autres moyens d'accéder aux éléments DOM, comme `document.querySelector()`, ne fonctionnent pas dans tous les environnements de test.

`TestElement` propose plusieurs méthodes pour interagir avec le DOM sous-jacent, comme `blur()`, `click()`, `getAttribute()`, et d'autres. Consultez la [page de référence de l'API TestElement](/api/cdk/testing/TestElement) pour la liste complète des méthodes.

N'exposez pas d'instances de `TestElement` aux utilisateurs du harnais, sauf s'il s'agit d'un élément que le consommateur du composant définit directement, comme le host element du composant. Exposer des instances de `TestElement` pour des éléments internes conduit les utilisateurs à dépendre de la structure DOM interne d'un composant.

Fournissez plutôt des méthodes plus ciblées pour les actions précises que l'utilisateur final peut effectuer ou les états particuliers qu'il peut observer. Par exemple, le `MyPopupHarness` des sections précédentes pourrait fournir des méthodes comme `toggle` et `isOpen` :

```ts
class MyPopupHarness extends ComponentHarness {
  static hostSelector = 'my-popup';

  protected getTriggerElement = this.locatorFor('button');
  protected getContentElement = this.locatorForOptional('.my-popup-content');

  /** Toggles the open state of the popup. */
  async toggle() {
    const trigger = await this.getTriggerElement();
    return trigger.click();
  }

  /** Checks if the popup us open. */
  async isOpen() {
    const content = await this.getContentElement();
    return !!content;
  }
}
```

## Charger des harnais pour les sous-composants {#loading-harnesses-for-subcomponents}

Les composants plus grands composent souvent des sous-composants. Vous pouvez aussi refléter cette structure dans le harnais d'un composant. Chacune des méthodes `locatorFor` de `ComponentHarness` possède une signature alternative qui permet de localiser des sous-harnais plutôt que des éléments.

Consultez la [page de référence de l'API ComponentHarness](/api/cdk/testing/ComponentHarness) pour la liste complète des différentes méthodes locatorFor.

Par exemple, considérez un menu construit à partir de la popup ci-dessus :

```ts
@Directive({
  selector: 'my-menu-item',
})
class MyMenuItem {}

@Component({
  selector: 'my-menu',
  template: `
    <my-popup>
      <ng-content />
    </my-popup>
  `,
})
class MyMenu {
  triggerText = input('');

  items = contentChildren(MyMenuItem);
}
```

Le harnais de `MyMenu` peut alors tirer parti d'autres harnais pour `MyPopup` et `MyMenuItem` :

```ts
class MyMenuHarness extends ComponentHarness {
  static hostSelector = 'my-menu';

  protected getPopupHarness = this.locatorFor(MyPopupHarness);

  /** Gets the currently shown menu items (empty list if menu is closed). */
  getItems = this.locatorForAll(MyMenuItemHarness);

  /** Toggles open state of the menu. */
  async toggle() {
    const popupHarness = await this.getPopupHarness();
    return popupHarness.toggle();
  }
}

class MyMenuItemHarness extends ComponentHarness {
  static hostSelector = 'my-menu-item';
}
```

## Filtrer les instances de harnais avec `HarnessPredicate` {#filtering-harness-instances-with-harnesspredicate}

Quand une page contient plusieurs instances d'un composant donné, vous pouvez vouloir filtrer selon une propriété du composant pour obtenir une instance précise. Par exemple, vous pouvez vouloir un bouton avec un texte précis, ou un menu avec un ID précis. La classe `HarnessPredicate` peut capturer ce type de critères pour une sous-classe de `ComponentHarness`. Même si l'auteur du test peut construire des instances de `HarnessPredicate` manuellement, c'est plus simple quand la sous-classe de `ComponentHarness` fournit une méthode utilitaire pour construire des prédicats pour les filtres courants.

Créez une méthode statique `with()` sur chaque sous-classe de `ComponentHarness`, qui renvoie un `HarnessPredicate` pour cette classe. Cela permet aux auteurs de tests d'écrire du code facile à comprendre, par exemple `loader.getHarness(MyMenuHarness.with({selector: '#menu1'}))`. En plus des options standard selector et ancestor, la méthode `with` doit ajouter toutes les autres options pertinentes pour la sous-classe concernée.

Les harnais qui ont besoin d'options supplémentaires doivent étendre l'interface `BaseHarnessFilters` et ajouter les propriétés optionnelles nécessaires. `HarnessPredicate` fournit plusieurs méthodes pratiques pour ajouter des options : `stringMatches()`, `addOption()` et `add()`. Consultez la [page de l'API HarnessPredicate](/api/cdk/testing/HarnessPredicate) pour la description complète.

Par exemple, avec un menu, il est utile de filtrer selon le texte du déclencheur et de filtrer les éléments de menu selon leur texte :

```ts
interface MyMenuHarnessFilters extends BaseHarnessFilters {
  /** Filters based on the trigger text for the menu. */
  triggerText?: string | RegExp;
}

interface MyMenuItemHarnessFilters extends BaseHarnessFilters {
  /** Filters based on the text of the menu item. */
  text?: string | RegExp;
}

class MyMenuHarness extends ComponentHarness {
  static hostSelector = 'my-menu';

  /** Creates a `HarnessPredicate` used to locate a particular `MyMenuHarness`. */
  static with(options: MyMenuHarnessFilters): HarnessPredicate<MyMenuHarness> {
    return new HarnessPredicate(MyMenuHarness, options).addOption(
      'trigger text',
      options.triggerText,
      (harness, text) => HarnessPredicate.stringMatches(harness.getTriggerText(), text),
    );
  }

  protected getPopupHarness = this.locatorFor(MyPopupHarness);

  /** Gets the text of the menu trigger. */
  async getTriggerText(): Promise<string> {
    const popupHarness = await this.getPopupHarness();
    return popupHarness.getTriggerText();
  }
}

class MyMenuItemHarness extends ComponentHarness {
  static hostSelector = 'my-menu-item';

  /** Creates a `HarnessPredicate` used to locate a particular `MyMenuItemHarness`. */
  static with(options: MyMenuItemHarnessFilters): HarnessPredicate<MyMenuItemHarness> {
    return new HarnessPredicate(MyMenuItemHarness, options).addOption(
      'text',
      options.text,
      (harness, text) => HarnessPredicate.stringMatches(harness.getText(), text),
    );
  }

  /** Gets the text of the menu item. */
  async getText(): Promise<string> {
    const host = await this.host();
    return host.text();
  }
}
```

Vous pouvez passer un `HarnessPredicate` au lieu d'une classe `ComponentHarness` à n'importe quelle API de `HarnessLoader`, `LocatorFactory` ou `ComponentHarness`. Cela permet aux auteurs de tests de cibler facilement une instance de composant précise lors de la création d'une instance de harnais. Cela permet aussi à l'auteur du harnais de réutiliser le même `HarnessPredicate` pour proposer des API plus puissantes sur sa classe de harnais. Par exemple, considérez la méthode `getItems` du `MyMenuHarness` présenté ci-dessus. Ajouter une API de filtrage permet aux utilisateurs du harnais de rechercher des éléments de menu précis :

```ts
class MyMenuHarness extends ComponentHarness {
  static hostSelector = 'my-menu';

  /** Gets a list of items in the menu, optionally filtered based on the given criteria. */
  async getItems(filters: MyMenuItemHarnessFilters = {}): Promise<MyMenuItemHarness[]> {
    const getFilteredItems = this.locatorForAll(MyMenuItemHarness.with(filters));
    return getFilteredItems();
  }
  ...
}
```

## Créer un `HarnessLoader` pour les éléments qui utilisent la projection de contenu {#creating-harnessloader-for-elements-that-use-content-projection}

Certains composants projettent du contenu supplémentaire dans le template du composant. Consultez le [guide de la projection de contenu](guide/components/content-projection) pour plus d'informations.

Ajoutez une instance de `HarnessLoader` limitée à l'élément contenant le `<ng-content>` quand vous créez un harnais pour un composant qui utilise la projection de contenu. Cela permet à l'utilisateur du harnais de charger des harnais supplémentaires pour les composants passés en contenu. `ComponentHarness` propose plusieurs méthodes pour créer des instances de HarnessLoader dans ce genre de cas : `harnessLoaderFor()`, `harnessLoaderForOptional()`, `harnessLoaderForAll()`. Consultez la [page de référence de l'API de l'interface HarnessLoader](/api/cdk/testing/HarnessLoader) pour plus de détails.

Par exemple, le `MyPopupHarness` présenté plus haut peut étendre `ContentContainerComponentHarness` pour permettre de charger des harnais dans le `<ng-content>` du composant.

```ts
class MyPopupHarness extends ContentContainerComponentHarness<string> {
  static hostSelector = 'my-popup';
}
```

## Accéder aux éléments en dehors du host element du composant {#accessing-elements-outside-of-the-components-host-element}

Il arrive qu'un harnais de composant doive accéder à des éléments situés en dehors du host element du composant correspondant. Par exemple, le code qui affiche un élément flottant ou une pop-up attache souvent des éléments DOM directement au body du document, comme le service `Overlay` du CDK Angular.

Dans ce cas, `ComponentHarness` fournit une méthode permettant d'obtenir une `LocatorFactory` pour l'élément racine du document. La `LocatorFactory` prend en charge la plupart des API de la classe de base `ComponentHarness` et peut ensuite servir à effectuer des recherches relatives à l'élément racine du document.

Supposons que le composant `MyPopup` ci-dessus utilise l'overlay du CDK pour le contenu de la popup, plutôt qu'un élément de son propre template. Dans ce cas, `MyPopupHarness` devrait accéder à l'élément de contenu via la méthode `documentRootLocatorFactory()`, qui récupère une locator factory dont la racine est la racine du document.

```ts
class MyPopupHarness extends ComponentHarness {
  static hostSelector = 'my-popup';

  /** Gets a `HarnessLoader` whose root element is the popup's content element. */
  async getHarnessLoaderForContent(): Promise<HarnessLoader> {
    const rootLocator = this.documentRootLocatorFactory();
    return rootLocator.harnessLoaderFor('my-popup-content');
  }
}
```

## Attendre les tâches asynchrones {#waiting-for-asynchronous-tasks}

Les méthodes de `TestElement` déclenchent automatiquement la détection de changements d'Angular et attendent les tâches dans la `NgZone`. Dans la plupart des cas, les auteurs de harnais n'ont rien de particulier à faire pour attendre les tâches asynchrones. Il existe toutefois quelques cas limites où cela peut ne pas suffire.

Dans certaines circonstances, les animations Angular peuvent nécessiter un second cycle de détection de changements, suivi d'une stabilisation de la `NgZone`, avant que les événements d'animation soient entièrement traités. Quand c'est nécessaire, `ComponentHarness` propose une méthode `forceStabilize()` que vous pouvez appeler pour effectuer ce second passage.

Vous pouvez utiliser `NgZone.runOutsideAngular()` pour planifier des tâches en dehors de NgZone. Appelez la méthode `waitForTasksOutsideAngular()` du harnais correspondant si vous devez attendre explicitement les tâches en dehors de `NgZone`, car cela ne se fait pas automatiquement.
