<!-- source: 5f5ff90e1daa6275a164158e47282cdce31aeb6e -->
# Migrer de Karma vers Vitest

L'Angular CLI utilise [Vitest](https://vitest.dev/) comme test runner unitaire par défaut pour les nouveaux projets. Ce guide explique comment migrer un projet existant de Karma et Jasmine vers Vitest.

IMPORTANT: La migration d'un projet existant vers Vitest est considérée comme expérimentale. Ce processus nécessite aussi d'utiliser le système de build `application`, utilisé par défaut dans tous les nouveaux projets.

## Étapes de migration manuelle {#manual-migration-steps}

Avant d'utiliser le schematic de refactoring automatisé, vous devez mettre à jour manuellement votre projet pour qu'il utilise le test runner Vitest.

### 1. Installer les dépendances {#1-install-dependencies}

Installez `vitest` et une library d'émulation du DOM. Même si les tests dans un navigateur restent possibles (voir l'[étape 5](#5-configure-browser-mode-optional)), Vitest utilise par défaut une library d'émulation du DOM pour simuler un environnement de navigateur dans Node.js et exécuter les tests plus rapidement. La CLI détecte et utilise automatiquement `happy-dom` s'il est installé ; sinon, elle se rabat sur `jsdom`. L'un de ces packages doit être installé.

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev vitest jsdom
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev vitest jsdom
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D vitest jsdom
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev vitest jsdom
  </docs-code>
</docs-code-multifile>

### 2. Mettre à jour `angular.json` {#2-update-angularjson}

Dans votre fichier `angular.json`, trouvez la cible `test` de votre projet et remplacez le `builder` par `@angular/build:unit-test`.

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test"
        }
      }
    }
  }
}
```

Le builder `unit-test` utilise par défaut `"tsConfig": "tsconfig.spec.json"` et `"buildTarget": "::development"`. Vous pouvez définir explicitement ces options si votre projet nécessite des valeurs différentes. Par exemple, si la configuration de build `development` est absente ou si vous avez besoin d'options différentes pour les tests, vous pouvez créer et utiliser une configuration de build `testing` (ou d'un nom similaire) pour `buildTarget`.

Le builder `@angular/build:karma` permettait auparavant de configurer des options de build (comme `polyfills`, `assets` ou `styles`) directement dans la cible `test`. Le nouveau builder `@angular/build:unit-test` ne le permet pas. Si vos options de build propres aux tests diffèrent de votre configuration de build `development` existante, vous devez les déplacer dans une configuration de cible de build dédiée. Si vos options de build de test correspondent déjà à votre configuration de build `development`, aucune action n'est nécessaire.

### 3. Gérer les configurations personnalisées de `karma.conf.js` {#3-handle-custom-karmaconfjs-configurations}

Les configurations personnalisées de `karma.conf.js` ne sont pas migrées automatiquement. Avant de supprimer votre fichier `karma.conf.js`, examinez-le pour repérer les réglages personnalisés à migrer.

De nombreuses options de Karma ont des équivalents dans Vitest, que vous pouvez définir dans un fichier de configuration Vitest personnalisé (par ex. `vitest.config.ts`) relié à votre `angular.json` via l'option `runnerConfig`.

Les cas de migration courants sont les suivants :

- **Reporters** : les reporters Karma doivent être remplacés par des reporters compatibles avec Vitest. Ils peuvent souvent être configurés directement dans votre `angular.json`, dans la propriété `test.options.reporters`. Pour des configurations plus avancées, utilisez un fichier `vitest.config.ts` personnalisé.
- **Plugins** : les plugins Karma peuvent avoir des équivalents Vitest qu'il vous faudra trouver et installer. Notez que la couverture de code est une fonctionnalité à part entière de l'Angular CLI et peut être activée avec `ng test --coverage`.
- **Launchers de navigateur personnalisés** : ils sont remplacés par l'option `browsers` dans `angular.json` et l'installation d'un provider de navigateur comme `@vitest/browser-playwright`.

Pour les autres réglages, consultez la [documentation officielle de Vitest](https://vitest.dev/config/).

### 4. Supprimer Karma ainsi que les fichiers `test.ts` {#4-remove-karma-and-testts-files}

Vous pouvez maintenant supprimer `karma.conf.js` et `src/test.ts` de votre projet et désinstaller les packages liés à Karma. Les commandes suivantes se basent sur les packages installés dans un nouveau projet Angular CLI ; votre projet peut contenir d'autres packages liés à Karma à supprimer.

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm uninstall karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn remove karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm remove karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core
  </docs-code>
  <docs-code header="bun" language="shell">
    bun remove karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core
  </docs-code>
</docs-code-multifile>

### 5. Configurer le mode navigateur (facultatif) {#5-configure-browser-mode-optional}

Si vous devez exécuter les tests dans un vrai navigateur, vous devez installer un provider de navigateur et configurer votre `angular.json`.

**Installer un provider de navigateur :**

Choisissez l'un des providers de navigateur suivants selon vos besoins :

- **Playwright** : `@vitest/browser-playwright` pour Chromium, Firefox et WebKit.
- **WebdriverIO** : `@vitest/browser-webdriverio` pour Chrome, Firefox, Safari et Edge.
- **Preview** : `@vitest/browser-preview` pour les environnements WebContainer (comme StackBlitz).

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev @vitest/browser-playwright
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev @vitest/browser-playwright
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D @vitest/browser-playwright
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev @vitest/browser-playwright
  </docs-code>
</docs-code-multifile>

**Mettre à jour `angular.json` pour le mode navigateur :**

Ajoutez l'option `browsers` aux options de votre cible `test`. Le nom du navigateur dépend du provider installé (par ex. `chromium` pour Playwright, `chrome` pour WebdriverIO).

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "browsers": ["chromium"]
          }
        }
      }
    }
  }
}
```

Le mode headless est activé automatiquement si la variable d'environnement `CI` est définie ou si le nom d'un navigateur contient « Headless » (par ex. `ChromeHeadless`). Sinon, les tests s'exécutent dans un navigateur avec interface (headed).

## Refactoring automatisé des tests avec les schematics {#automated-test-refactoring-with-schematics}

IMPORTANT: Le schematic `refactor-jasmine-vitest` est expérimental et peut ne pas couvrir tous les patterns de test possibles. Vérifiez toujours les modifications apportées par le schematic.

L'Angular CLI fournit le schematic `refactor-jasmine-vitest` pour refactoriser automatiquement vos tests Jasmine afin qu'ils utilisent Vitest.

### Vue d'ensemble {#overview}

Le schematic automatise les transformations suivantes dans vos fichiers de test (`.spec.ts`) :

- Convertit `fit` et `fdescribe` en `it.only` et `describe.only`.
- Convertit `xit` et `xdescribe` en `it.skip` et `describe.skip`.
- Convertit les appels à `spyOn` en leur équivalent `vi.spyOn`.
- Remplace `jasmine.objectContaining` par `expect.objectContaining`.
- Remplace `jasmine.any` par `expect.any`.
- Remplace `jasmine.createSpy` par `vi.fn`.
- Met à jour `beforeAll`, `beforeEach`, `afterAll` et `afterEach` vers leurs équivalents Vitest.
- Convertit `fail()` en `expect.fail()` de Vitest.
- Ajuste les attentes pour correspondre aux API de Vitest
- Ajoute des commentaires TODO pour le code qui ne peut pas être converti automatiquement

Le schematic **n'effectue pas** les actions suivantes :

- Il n'installe pas `vitest` ni les autres dépendances associées.
- Il ne modifie pas votre `angular.json` pour utiliser le builder Vitest et ne migre aucune option de build (comme `polyfills` ou `styles`) depuis la cible `test`.
- Il ne supprime pas les fichiers `karma.conf.js` ou `test.ts`.
- Il ne gère pas les scénarios de spies complexes ou imbriqués, qui peuvent nécessiter un refactoring manuel.

### Exécuter le schematic {#running-the-schematic}

Une fois votre projet configuré pour Vitest, vous pouvez exécuter le schematic pour refactoriser vos fichiers de test.

Pour refactoriser **tous** les fichiers de test de votre projet par défaut, exécutez :

```bash
ng g @schematics/angular:refactor-jasmine-vitest
```

### Options {#options}

Vous pouvez utiliser les options suivantes pour personnaliser le comportement du schematic :

| Option                   | Description                                                                                         |
| :----------------------- | :-------------------------------------------------------------------------------------------------- |
| `--project <name>`       | Indique le projet à refactoriser dans un workspace multi-projets. <br> Exemple : `--project=my-lib` |
| `--include <path>`       | Refactorise uniquement un fichier ou un répertoire précis. <br> Exemple : `--include=src/app/app.component.spec.ts` |
| `--file-suffix <suffix>` | Indique un autre suffixe de fichier pour les fichiers de test. <br> Exemple : `--file-suffix=.test.ts` |
| `--add-imports`          | Ajoute des imports `vitest` explicites si vous avez désactivé les globals dans votre configuration Vitest. |
| `--verbose`              | Affiche un journal détaillé de toutes les transformations appliquées.                               |
| `--browser-mode`         | Si vous prévoyez d'exécuter les tests en mode navigateur.                                           |

### Après la migration {#after-migrating}

Une fois le schematic terminé, il est recommandé de :

1.  **Exécuter vos tests** : lancez `ng test` pour vous assurer que tous les tests passent toujours après le refactoring.
2.  **Vérifier les modifications** : examinez les modifications apportées par le schematic, en prêtant une attention particulière aux tests complexes, notamment ceux qui comportent des spies ou des mocks élaborés, car ils peuvent nécessiter d'autres ajustements manuels.

La commande `ng test` compile l'application en _mode watch_ et lance le runner configuré. Le mode watch est activé par défaut quand vous utilisez un terminal interactif et que l'exécution n'a pas lieu en CI.

## Configuration {#configuration}

L'Angular CLI se charge pour vous de la configuration de Vitest, en construisant la configuration complète en mémoire à partir des options de `angular.json`.

### Configuration Vitest personnalisée {#custom-vitest-configuration}

IMPORTANT: Si une configuration personnalisée donne accès à des options avancées, l'équipe Angular n'assure aucun support direct pour le contenu spécifique du fichier de configuration ni pour les plugins tiers qui y sont utilisés. La CLI écrase aussi certaines propriétés (`test.projects`, `test.include`) pour garantir un fonctionnement correct.

Vous pouvez fournir un fichier de configuration Vitest personnalisé pour remplacer les réglages par défaut. Pour la liste complète des options disponibles, consultez la [documentation officielle de Vitest](https://vitest.dev/config/).

**1. Chemin direct :**
Indiquez un chemin direct vers un fichier de configuration Vitest dans votre `angular.json` :

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {"runnerConfig": "vitest.config.ts"}
        }
      }
    }
  }
}
```

**2. Recherche automatique d'une configuration de base :**
Si vous définissez `runnerConfig` à `true`, le builder recherche automatiquement un fichier partagé `vitest-base.config.*` à la racine de votre projet et de votre workspace.

## Patch Vitest de `zone.js` {#zonejs-vitest-patch}

Pour utiliser des fonctions comme `fakeAsync`, `flush` ou `waitForAsync`, ou pour permettre à vos tests existants de fonctionner avec elles, vous pouvez ajouter `zone.js/plugins/vitest-patch` aux polyfills de votre cible de test dans `angular.json`.

Quoi qu'il en soit, nous vous recommandons vivement de commencer à planifier la conversion de vos suites de tests existantes vers l'`async` natif et les fake timers de Vitest, car c'est l'approche établie.

Consultez [cet exemple](/guide/testing/components-scenarios#async-test-with-a-vitest-fake-timers) d'utilisation des fake timers avec Vitest.

## Signaler des bugs {#bug-reports}

Signalez les problèmes et les demandes de fonctionnalités sur [GitHub](https://github.com/angular/angular-cli/issues).

Dans la mesure du possible, fournissez une reproduction minimale pour aider l'équipe à traiter les problèmes.
