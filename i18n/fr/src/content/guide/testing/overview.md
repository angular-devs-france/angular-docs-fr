<!-- source: 00b4c1d7fc6e738739ad8d70c2cd3147358e3016 -->
# Tests unitaires

Tester votre application Angular vous aide à vérifier qu'elle fonctionne comme prévu. Les tests unitaires sont essentiels pour détecter les bugs tôt, garantir la qualité du code et faciliter un refactoring sans risque.

NOTE: Ce guide couvre la configuration de test par défaut des nouveaux projets Angular CLI, qui utilise Vitest. Si vous migrez un projet existant depuis Karma, consultez le [guide de migration de Karma vers Vitest](guide/testing/migrating-to-vitest). Karma reste pris en charge ; pour plus d'informations, consultez le [guide de test avec Karma](guide/testing/karma).

## Configuration pour les tests {#set-up-for-testing}

L'Angular CLI télécharge et installe tout ce dont vous avez besoin pour tester une application Angular avec le [framework de test Vitest](https://vitest.dev). Les nouveaux projets incluent `vitest` et `jsdom` par défaut.

Vitest exécute vos tests unitaires dans un environnement Node.js. Pour simuler le DOM du navigateur, Vitest utilise une library appelée `jsdom`. L'exécution des tests est ainsi plus rapide, car elle évite le coût du lancement d'un navigateur. Vous pouvez remplacer `jsdom` par une alternative comme `happy-dom` en l'installant et en désinstallant `jsdom`. Actuellement, `jsdom` et `happy-dom` sont les libraries d'émulation du DOM prises en charge.

Le projet que vous créez avec la CLI est immédiatement prêt à être testé. Exécutez la commande [`ng test`](cli/test) :

```shell
ng test
```

La commande `ng test` compile l'application en _mode watch_ et lance le [test runner Vitest](https://vitest.dev).

La sortie de la console ressemble à ceci :

```shell
 ✓ src/app/app.spec.ts (2)
   ✓ App should create the app
   ✓ App should render title
 Test Files  1 passed (1)
      Tests  2 passed (2)
   Start at  18:18:01
   Duration  2.46s (transform 615ms, setup 2ms, collect 2.21s, tests 5ms)
```

La commande `ng test` surveille aussi les modifications de vos fichiers. Si vous modifiez un fichier et l'enregistrez, les tests s'exécutent à nouveau.

## Configuration {#configuration}

L'Angular CLI gère pour vous l'essentiel de la configuration de Vitest. Vous pouvez personnaliser le comportement des tests en modifiant les options de la cible `test` dans votre fichier `angular.json`.

### Options d'angular.json {#angularjson-options}

- `include` : patterns glob des fichiers à inclure dans les tests. Par défaut : `['**/*.spec.ts', '**/*.test.ts']`.
- `exclude` : patterns glob des fichiers à exclure des tests.
- `setupFiles` : liste de chemins vers des fichiers de configuration globale (par ex. des polyfills ou des mocks globaux) exécutés avant vos tests.
- `providersFile` : chemin vers un fichier qui exporte par défaut un tableau de providers Angular pour l'environnement de test. C'est utile pour configurer des providers de test globaux, injectés dans vos tests.
- `coverage` : booléen qui active ou désactive le rapport de couverture de code. Par défaut : `false`.
- `browsers` : tableau de noms de navigateurs pour exécuter les tests dans un vrai navigateur (par ex. `["chromium"]`). Nécessite l'installation d'un provider de navigateur. Consultez la section [Exécuter les tests dans un navigateur](#running-tests-in-a-browser) pour plus de détails.

### Configuration et providers de test globaux {#global-test-setup-and-providers}

Les options `setupFiles` et `providersFile` sont particulièrement utiles pour gérer la configuration de test globale.

Par exemple, vous pouvez créer un fichier `src/test-providers.ts` pour fournir `provideHttpClientTesting` à tous vos tests :

```typescript {header: "src/test-providers.ts"}
import {EnvironmentProviders, Provider} from '@angular/core';
import {provideHttpClientTesting} from '@angular/common/http/testing';

const testProviders: (Provider | EnvironmentProviders)[] = [provideHttpClientTesting()];

export default testProviders;
```

Vous référencez ensuite ce fichier dans votre `angular.json` :

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "providersFile": "src/test-providers.ts"
          }
        }
      }
    }
  }
}
```

HELPFUL: Quand vous créez de nouveaux fichiers TypeScript pour la configuration des tests ou les providers, comme `src/test-providers.ts`, assurez-vous qu'ils sont inclus dans le fichier de configuration TypeScript des tests de votre projet (généralement `tsconfig.spec.json`). Le compilateur TypeScript peut ainsi traiter correctement ces fichiers pendant les tests.

### Configuration avancée de Vitest {#advanced-vitest-configuration}

Pour les cas d'usage avancés, vous pouvez fournir un fichier de configuration Vitest personnalisé avec l'option `runnerConfig` dans `angular.json`.

IMPORTANT: Si une configuration personnalisée donne accès à des options avancées, l'équipe Angular n'assure aucun support pour le contenu du fichier de configuration ni pour les plugins tiers. La CLI écrase aussi certaines propriétés (`test.projects`, `test.include`) pour garantir une intégration correcte.

Vous pouvez créer un fichier de configuration Vitest (par ex. `vitest-base.config.ts`) et le référencer dans votre `angular.json` :

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "runnerConfig": "vitest-base.config.ts"
          }
        }
      }
    }
  }
}
```

Vous pouvez aussi générer un fichier de configuration de base avec la CLI :

```shell
ng generate config vitest
```

Cela crée un fichier `vitest-base.config.ts` que vous pouvez personnaliser.

HELPFUL: Pour en savoir plus sur la configuration de Vitest, consultez la [documentation officielle de Vitest](https://vitest.dev/config/).

## Couverture de code {#code-coverage}

Vous pouvez générer un rapport de couverture de code en ajoutant l'option `--coverage` à la commande `ng test`. Le rapport est généré dans le répertoire `coverage/`.

Pour des informations plus détaillées, consultez le [guide de la couverture de code](guide/testing/code-coverage).

## Exécuter les tests dans un navigateur {#running-tests-in-a-browser}

Si l'environnement Node.js par défaut est plus rapide pour la plupart des tests unitaires, vous pouvez aussi exécuter vos tests dans un vrai navigateur. C'est utile pour les tests qui reposent sur des API propres au navigateur (comme le rendu) ou pour le débogage.

Pour exécuter les tests dans un navigateur, vous devez d'abord installer un provider de navigateur. Pour en savoir plus sur le mode navigateur de Vitest, consultez la [documentation officielle](https://vitest.dev/guide/browser).

Une fois le provider installé, vous pouvez exécuter vos tests dans le navigateur en configurant l'option `browsers` dans `angular.json` ou en utilisant l'option CLI `--browsers`. Par défaut, les tests s'exécutent dans un navigateur avec interface (headed). Si la variable d'environnement `CI` est définie, le mode headless est utilisé à la place. Pour contrôler explicitement le mode headless, vous pouvez ajouter le suffixe `Headless` au nom du navigateur (par ex. `chromiumHeadless`).

```bash
# Example for Playwright (headed)
ng test --browsers=chromium

# Example for Playwright (headless)
ng test --browsers=chromiumHeadless

# Example for WebdriverIO (headed)
ng test --browsers=chrome

# Example for WebdriverIO (headless)
ng test --browsers=chromeHeadless
```

Choisissez l'un des providers de navigateur suivants selon vos besoins :

### Playwright {#playwright}

[Playwright](https://playwright.dev/) est une library d'automatisation de navigateur qui prend en charge Chromium, Firefox et WebKit.

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev @vitest/browser-playwright playwright
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev @vitest/browser-playwright playwright
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D @vitest/browser-playwright playwright
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev @vitest/browser-playwright playwright
  </docs-code>
</docs-code-multifile>

### WebdriverIO {#webdriverio}

[WebdriverIO](https://webdriver.io/) est un framework de test d'automatisation pour navigateurs et mobiles qui prend en charge Chrome, Firefox, Safari et Edge.

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev @vitest/browser-webdriverio webdriverio
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev @vitest/browser-webdriverio webdriverio
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D @vitest/browser-webdriverio webdriverio
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev @vitest/browser-webdriverio webdriverio
  </docs-code>
</docs-code-multifile>

### Preview {#preview}

Le provider `@vitest/browser-preview` est conçu pour les environnements WebContainer comme StackBlitz et n'est pas destiné à un usage en CI/CD.

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev @vitest/browser-preview
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev @vitest/browser-preview
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D @vitest/browser-preview
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev @vitest/browser-preview
  </docs-code>
</docs-code-multifile>

HELPFUL: Pour une configuration plus avancée propre au navigateur, consultez la section [Configuration avancée de Vitest](#advanced-vitest-configuration).

## Autres frameworks de test {#other-test-frameworks}

Vous pouvez aussi écrire les tests unitaires d'une application Angular avec d'autres libraries de test et d'autres test runners. Chaque library et chaque runner a ses propres procédures d'installation, sa configuration et sa syntaxe.

## Tester en intégration continue {#testing-in-continuous-integration}

Une suite de tests robuste est un élément clé d'un pipeline d'intégration continue (CI). Les serveurs de CI vous permettent d'automatiser l'exécution de vos tests à chaque commit et à chaque pull request.

Pour tester votre application Angular sur un serveur de CI, exécutez la commande de test standard :

```shell
ng test
```

La plupart des serveurs de CI définissent une variable d'environnement `CI=true`, que `ng test` détecte. Vos tests sont alors automatiquement configurés pour s'exécuter en mode non interactif, en une seule passe.

Si votre serveur de CI ne définit pas cette variable, ou si vous devez forcer manuellement l'exécution en une seule passe, vous pouvez utiliser les options `--no-watch` et `--no-progress` :

```shell
ng test --no-watch --no-progress
```

## Plus d'informations sur les tests {#more-information-on-testing}

Une fois votre application configurée pour les tests, les guides de test suivants pourraient vous être utiles.

|                                                                    | Détails                                                                           |
| :----------------------------------------------------------------- | :-------------------------------------------------------------------------------- |
| [Couverture de code](guide/testing/code-coverage)                  | La part de votre application couverte par vos tests et comment exiger un seuil minimal. |
| [Tester les services](guide/testing/services)                      | Comment tester les services utilisés par votre application.                       |
| [Bases du test de composants](guide/testing/components-basics)     | Les bases du test des composants Angular.                                         |
| [Scénarios de test de composants](guide/testing/components-scenarios) | Différents scénarios et cas d'usage du test de composants.                     |
| [Tester les directives d'attribute](guide/testing/attribute-directives) | Comment tester vos directives d'attribute.                                   |
| [Tester les pipes](guide/testing/pipes)                            | Comment tester les pipes.                                                         |
| [Déboguer les tests](guide/testing/debugging)                      | Les bugs de test courants.                                                        |
| [API utilitaires de test](guide/testing/utility-apis)              | Les fonctionnalités de test d'Angular.                                            |
