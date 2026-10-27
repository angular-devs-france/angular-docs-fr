<!-- source: ada708ab62f017b3a12db550397a17561ec65fa1 -->
# Prendre en charge les harnais dans d'autres environnements de test

## Avant de commencer {#before-you-start}

TIP: Ce guide suppose que vous avez déjà lu la [vue d'ensemble des harnais de composant](guide/testing/component-harnesses-overview). Lisez-la d'abord si vous découvrez les harnais de composant.

### Quand est-il pertinent d'ajouter la prise en charge d'un environnement de test ? {#when-does-adding-support-for-a-test-environment-make-sense}

Pour utiliser les harnais de composant dans les environnements suivants, vous pouvez utiliser les deux environnements intégrés du CDK Angular :

- Tests unitaires
- Tests de bout en bout WebDriver

Pour utiliser un environnement de test pris en charge, lisez le guide [Créer des harnais pour vos composants](guide/testing/creating-component-harnesses).

Sinon, pour prendre en charge d'autres environnements, vous devez définir comment interagir avec un élément DOM et comment fonctionnent les interactions DOM dans votre environnement. Poursuivez votre lecture pour en savoir plus.

### Installation du CDK {#cdk-installation}

Le [Component Dev Kit (CDK)](https://material.angular.dev/cdk/categories) est un ensemble de primitives de comportement pour construire des composants. Pour utiliser les harnais de composant, installez d'abord `@angular/cdk` depuis npm. Vous pouvez le faire depuis votre terminal avec l'Angular CLI :

```shell
ng add @angular/cdk
```

## Créer une implémentation de `TestElement` {#creating-a-testelement-implementation}

Chaque environnement de test doit définir une implémentation de `TestElement`. L'interface `TestElement` sert de représentation d'un élément DOM indépendante de l'environnement. Elle permet aux harnais d'interagir avec les éléments DOM quel que soit l'environnement sous-jacent. Comme certains environnements ne permettent pas d'interagir de manière synchrone avec les éléments DOM (par exemple WebDriver), toutes les méthodes de `TestElement` sont asynchrones et renvoient une `Promise` contenant le résultat de l'opération.

`TestElement` propose plusieurs méthodes pour interagir avec le DOM sous-jacent, comme `blur()`, `click()`, `getAttribute()`, et d'autres. Consultez la [page de référence de l'API TestElement](/api/cdk/testing/TestElement) pour la liste complète des méthodes.

L'interface `TestElement` se compose en grande partie de méthodes qui ressemblent à celles disponibles sur `HTMLElement`. Des méthodes similaires existent dans la plupart des environnements de test, ce qui rend leur implémentation assez simple. Une différence importante est toutefois à noter lors de l'implémentation de la méthode `sendKeys` : les codes de touche de l'enum `TestKey` diffèrent probablement de ceux utilisés dans l'environnement de test. Les auteurs d'environnements doivent maintenir une correspondance entre les codes `TestKey` et les codes utilisés dans l'environnement de test concerné.

Les implémentations [UnitTestElement](/api/cdk/testing/testbed/UnitTestElement) et [SeleniumWebDriverElement](/api/cdk/testing/selenium-webdriver/SeleniumWebDriverElement) du CDK Angular sont de bons exemples d'implémentation de cette interface.

## Créer une implémentation de `HarnessEnvironment` {#creating-a-harnessenvironment-implementation}

Les auteurs de tests utilisent `HarnessEnvironment` pour créer des instances de harnais de composant à utiliser dans les tests. `HarnessEnvironment` est une classe abstraite qui doit être étendue pour créer une sous-classe concrète pour le nouvel environnement. Pour prendre en charge un nouvel environnement de test, créez une sous-classe de `HarnessEnvironment` qui fournit des implémentations concrètes pour tous les membres abstraits.

`HarnessEnvironment` a un paramètre de type générique : `HarnessEnvironment<E>`. Ce paramètre, `E`, représente le type d'élément brut de l'environnement. Par exemple, ce paramètre vaut Element pour les environnements de tests unitaires.

Voici les méthodes abstraites à implémenter :

| Méthode | Description |
| :----------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `abstract getDocumentRoot(): E` | Récupère l'élément racine de l'environnement (par exemple `document.body`). |
| `abstract createTestElement(element: E): TestElement` | Crée un `TestElement` pour l'élément brut donné. |
| `abstract createEnvironment(element: E): HarnessEnvironment` | Crée un `HarnessEnvironment` dont la racine est l'élément brut donné. |
| `abstract getAllRawElements(selector: string): Promise<E[]>` | Récupère tous les éléments bruts sous l'élément racine de l'environnement qui correspondent au selector donné. |
| `abstract forceStabilize(): Promise<void>` | Récupère une `Promise` qui se résout quand `NgZone` est stable. En outre, le cas échéant, demande à `NgZone` de se stabiliser (par exemple en appelant `flush()` dans un test `fakeAsync`). |
| `abstract waitForTasksOutsideAngular(): Promise<void>` | Récupère une `Promise` qui se résout quand la zone parente de `NgZone` est stable. |

En plus d'implémenter les méthodes manquantes, cette classe doit fournir aux auteurs de tests un moyen d'obtenir des instances de `ComponentHarness`. Définissez un constructeur protégé et fournissez une méthode statique appelée `loader` qui renvoie une instance de `HarnessLoader`. Cela permet aux auteurs de tests d'écrire du code comme : `SomeHarnessEnvironment.loader().getHarness(...)`. Selon les besoins de l'environnement concerné, la classe peut fournir plusieurs méthodes statiques différentes ou exiger des arguments. (Par exemple, la méthode `loader` de `TestbedHarnessEnvironment` prend une `ComponentFixture`, et la classe fournit des méthodes statiques supplémentaires appelées `documentRootLoader` et `harnessForFixture`.)

Les implémentations [`TestbedHarnessEnvironment`](/api/cdk/testing/testbed/TestbedHarnessEnvironment) et [SeleniumWebDriverHarnessEnvironment](/api/cdk/testing/selenium-webdriver/SeleniumWebDriverHarnessEnvironment) du CDK Angular sont de bons exemples d'implémentation de cette interface.

## Gérer la détection de changements automatique {#handling-auto-change-detection}

Pour prendre en charge `manualChangeDetection` et les API parallèles, votre environnement doit installer un handler pour le statut de la détection de changements automatique.

Quand votre environnement veut commencer à gérer le statut de la détection de changements automatique, il peut appeler `handleAutoChangeDetectionStatus(handler)`. La fonction handler reçoit un `AutoChangeDetectionStatus` qui possède deux propriétés : `isDisabled` et `onDetectChangesNow()`. Consultez la [page de référence de l'API AutoChangeDetectionStatus](/api/cdk/testing/AutoChangeDetectionStatus) pour plus d'informations.
Si votre environnement veut arrêter de gérer le statut de la détection de changements automatique, il peut appeler `stopHandlingAutoChangeDetectionStatus()`.
