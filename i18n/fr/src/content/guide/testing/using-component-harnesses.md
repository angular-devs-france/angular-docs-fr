<!-- source: 04040d7150a67784ac214d528e5ce19faf7d65c7 -->
# Utiliser des harnais de composant dans les tests

## Avant de commencer {#before-you-start}

TIP: Ce guide suppose que vous avez déjà lu la [vue d'ensemble des harnais de composant](guide/testing/component-harnesses-overview). Lisez-la d'abord si vous découvrez les harnais de composant.

### Installation du CDK {#cdk-installation}

Le [Component Dev Kit (CDK)](https://material.angular.dev/cdk/categories) est un ensemble de primitives de comportement pour construire des composants. Pour utiliser les harnais de composant, installez d'abord `@angular/cdk` depuis npm. Vous pouvez le faire depuis votre terminal avec l'Angular CLI :

```shell
ng add @angular/cdk
```

## Environnements et loaders de harnais de test {#test-harness-environments-and-loaders}

Vous pouvez utiliser les harnais de test de composant dans différents environnements de test. Le CDK Angular prend en charge deux environnements intégrés :

- Les tests unitaires avec le `TestBed` d'Angular
- Les tests de bout en bout avec [WebDriver](https://developer.mozilla.org/en-US/docs/Web/WebDriver)

Chaque environnement fournit un <strong>loader de harnais</strong>. Le loader crée les instances de harnais que vous utilisez dans vos tests. Vous trouverez ci-dessous des indications plus précises sur les environnements de test pris en charge.

Les autres environnements de test nécessitent des bindings personnalisés. Consultez le guide [Prendre en charge les harnais dans d'autres environnements de test](guide/testing/component-harnesses-testing-environments) pour plus d'informations.

### Utiliser le loader de `TestbedHarnessEnvironment` pour les tests unitaires {#using-the-loader-from-testbedharnessenvironment-for-unit-tests}

Pour les tests unitaires, vous pouvez créer un loader de harnais à partir de [TestbedHarnessEnvironment](/api/cdk/testing/testbed/TestbedHarnessEnvironment). Cet environnement utilise une [fixture de composant](api/core/testing/ComponentFixture) créée par le `TestBed` d'Angular.

Pour créer un loader de harnais dont la racine est l'élément racine de la fixture, utilisez la méthode `loader()` :

```ts
const fixture = TestBed.createComponent(MyComponent);

// Create a harness loader from the fixture
const loader = TestbedHarnessEnvironment.loader(fixture);
...

// Use the loader to get harness instances
const myComponentHarness = await loader.getHarness(MyComponent);
```

Pour créer un loader de harnais destiné aux harnais d'éléments situés en dehors de la fixture, utilisez la méthode `documentRootLoader()`. Par exemple, le code qui affiche un élément flottant ou une pop-up attache souvent des éléments DOM directement au body du document, comme le service `Overlay` du CDK Angular.

Vous pouvez aussi créer directement un harnais pour l'élément racine de cette fixture avec `harnessForFixture()`.

### Utiliser le loader de `SeleniumWebDriverHarnessEnvironment` pour les tests de bout en bout {#using-the-loader-from-seleniumwebdriverharnessenvironment-for-end-to-end-tests}

Pour les tests de bout en bout basés sur WebDriver, vous pouvez créer un loader de harnais avec `SeleniumWebDriverHarnessEnvironment`.

Utilisez la méthode `loader()` pour obtenir l'instance du loader de harnais pour le document HTML actuel, dont la racine est l'élément racine du document. Cet environnement utilise un client WebDriver.

```ts
let wd: webdriver.WebDriver = getMyWebDriverClient();
const loader = SeleniumWebDriverHarnessEnvironment.loader(wd);
...
const myComponentHarness = await loader.getHarness(MyComponent);
```

## Utiliser un loader de harnais {#using-a-harness-loader}

Les instances de loader de harnais correspondent à un élément DOM précis et servent à créer des instances de harnais de composant pour les éléments situés sous cet élément.

Pour obtenir le `ComponentHarness` de la première instance de l'élément, utilisez la méthode `getHarness()`. Pour obtenir toutes les instances de `ComponentHarness`, utilisez la méthode `getAllHarnesses()`.

```ts
// Get harness for first instance of the element
const myComponentHarness = await loader.getHarness(MyComponent);

// Get harnesses for all instances of the element
const myComponentHarnesses = await loader.getAllHarnesses(MyComponent);
```

En plus de `getHarness` et `getAllHarnesses`, `HarnessLoader` propose plusieurs autres méthodes utiles pour rechercher des harnais :

- `getHarnessAtIndex(...)` : récupère le harnais d'un composant qui correspond aux critères donnés, à un index précis.
- `countHarnesses(...)` : compte le nombre d'instances de composant qui correspondent aux critères donnés.
- `hasHarness(...)` : vérifie si au moins une instance de composant correspond aux critères donnés.

Par exemple, considérez un composant réutilisable dialog-button qui ouvre une boîte de dialogue au clic. Il contient les composants suivants, chacun avec un harnais correspondant :

- `MyDialogButton` (compose `MyButton` et `MyDialog` avec une API pratique)
- `MyButton` (un composant bouton standard)
- `MyDialog` (une boîte de dialogue ajoutée à `document.body` par `MyDialogButton` lors du clic)

Le test suivant charge les harnais de chacun de ces composants :

```ts
let fixture: ComponentFixture<MyDialogButton>;
let loader: HarnessLoader;
let rootLoader: HarnessLoader;

beforeEach(() => {
  fixture = TestBed.createComponent(MyDialogButton);
  loader = TestbedHarnessEnvironment.loader(fixture);
  rootLoader = TestbedHarnessEnvironment.documentRootLoader(fixture);
});

it('loads harnesses', async () => {
  // Load a harness for the bootstrapped component with `harnessForFixture`
  dialogButtonHarness = await TestbedHarnessEnvironment.harnessForFixture(
    fixture,
    MyDialogButtonHarness,
  );

  // The button element is inside the fixture's root element, so we use `loader`.
  const buttonHarness = await loader.getHarness(MyButtonHarness);

  // Click the button to open the dialog
  await buttonHarness.click();

  // The dialog is appended to `document.body`, outside of the fixture's root element,
  // so we use `rootLoader` in this case.
  const dialogHarness = await rootLoader.getHarness(MyDialogHarness);

  // ... make some assertions
});
```

### Comportement des harnais selon les environnements {#harness-behavior-in-different-environments}

Les harnais peuvent ne pas se comporter exactement de la même manière dans tous les environnements. Certaines différences sont inévitables entre une véritable interaction utilisateur et les événements simulés générés dans les tests unitaires. Le CDK Angular fait de son mieux pour harmoniser le comportement autant que possible.

### Interagir avec les éléments enfants {#interacting-with-child-elements}

Pour interagir avec les éléments situés sous l'élément racine de ce loader de harnais, utilisez l'instance de `HarnessLoader` d'un élément enfant. Pour la première instance de l'élément enfant, utilisez la méthode `getChildLoader()`. Pour toutes les instances de l'élément enfant, utilisez la méthode `getAllChildLoaders()`.

```ts
const myComponentHarness = await loader.getHarness(MyComponent);

// Get loader for first instance of child element with '.child' selector
const childLoader = await loader.getChildLoader('.child');

// Get loaders for all instances of child elements with '.child' selector
const allChildLoaders = await loader.getAllChildLoaders('.child');
```

### Filtrer les harnais {#filtering-harnesses}

Quand une page contient plusieurs instances d'un composant donné, vous pouvez vouloir filtrer selon une propriété du composant pour obtenir une instance précise. Pour cela, vous pouvez utiliser un <strong>prédicat de harnais</strong>, une classe qui associe une classe `ComponentHarness` à des fonctions prédicats permettant de filtrer les instances de composant.

Quand vous demandez un harnais à un `HarnessLoader`, vous fournissez en réalité une HarnessQuery. Une query peut être l'une de ces deux choses :

- Un constructeur de harnais, qui récupère simplement ce harnais
- Un `HarnessPredicate`, qui récupère les harnais filtrés selon une ou plusieurs conditions

`HarnessPredicate` prend en charge quelques filtres de base (selector, ancêtre) qui fonctionnent sur tout ce qui étend `ComponentHarness`.

```ts
// Example of loading a MyButtonComponentHarness with a harness predicate
const disabledButtonPredicate = new HarnessPredicate(MyButtonComponentHarness, {
  selector: '[disabled]',
});
const disabledButton = await loader.getHarness(disabledButtonPredicate);
```

Il est toutefois courant que les harnais implémentent une méthode statique `with()` qui accepte des options de filtrage propres au composant et renvoie un `HarnessPredicate`.

```ts
// Example of loading a MyButtonComponentHarness with a specific selector
const button = await loader.getHarness(MyButtonComponentHarness.with({selector: 'btn'}));
```

Pour plus de détails, reportez-vous à la documentation du harnais concerné, car les options de filtrage supplémentaires sont propres à chaque implémentation de harnais.

## Utiliser les API des harnais de test {#using-test-harness-apis}

Bien que chaque harnais définisse une API propre au composant correspondant, ils partagent tous une classe de base commune, [ComponentHarness](/api/cdk/testing/ComponentHarness). Cette classe de base définit une propriété statique, `hostSelector`, qui associe la classe de harnais aux instances du composant dans le DOM.

Au-delà de cela, l'API d'un harnais donné est propre au composant correspondant ; reportez-vous à la documentation du composant pour savoir comment utiliser un harnais précis.

Par exemple, voici un test pour un composant qui utilise le [harnais du composant slider d'Angular Material](https://material.angular.dev/components/slider/api#MatSliderHarness) :

```ts
it('should get value of slider thumb', async () => {
  const slider = await loader.getHarness(MatSliderHarness);
  const thumb = await slider.getEndThumb();
  expect(await thumb.getValue()).toBe(50);
});
```

## Interopérabilité avec la détection de changements d'Angular {#interop-with-angular-change-detection}

Par défaut, les harnais de test exécutent la [détection de changements](/best-practices/runtime-performance) d'Angular avant de lire l'état d'un élément DOM et après avoir interagi avec un élément DOM.

Vous pouvez parfois avoir besoin d'un contrôle plus fin de la détection de changements dans vos tests, par exemple pour vérifier l'état d'un composant pendant qu'une opération asynchrone est en cours. Dans ces cas, utilisez la fonction `manualChangeDetection` pour désactiver la gestion automatique de la détection de changements pour un bloc de code.

```ts
it('checks state while async action is in progress', async () => {
  const buttonHarness = await loader.getHarness(MyButtonHarness);
  await manualChangeDetection(async () => {
    await buttonHarness.click();
    fixture.detectChanges();
    // Check expectations while async click operation is in progress.
    expect(isProgressSpinnerVisible()).toBe(true);
    await fixture.whenStable();
    // Check expectations after async click operation complete.
    expect(isProgressSpinnerVisible()).toBe(false);
  });
});
```

Presque toutes les méthodes des harnais sont asynchrones et renvoient une `Promise` pour permettre :

- La prise en charge des tests unitaires
- La prise en charge des tests de bout en bout
- L'isolation des tests vis-à-vis des changements de comportement asynchrone

L'équipe Angular recommande d'utiliser [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) pour améliorer la lisibilité des tests. Appeler `await` bloque l'exécution de votre test jusqu'à ce que la `Promise` associée se résolve.

Il arrive que vous vouliez effectuer plusieurs actions simultanément et attendre qu'elles soient toutes terminées plutôt que d'effectuer chaque action l'une après l'autre. Par exemple, lire plusieurs propriétés d'un même composant. Dans ces situations, utilisez la fonction `parallel` pour paralléliser les opérations. La fonction parallel fonctionne de manière similaire à `Promise.all`, tout en optimisant les vérifications de détection de changements.

```ts
it('reads properties in parallel', async () => {
  const checkboxHarness = await loader.getHarness(MyCheckboxHarness);
  // Read the checked and intermediate properties simultaneously.
  const [checked, indeterminate] = await parallel(() => [
    checkboxHarness.isChecked(),
    checkboxHarness.isIndeterminate(),
  ]);
  expect(checked).toBe(false);
  expect(indeterminate).toBe(true);
});
```
