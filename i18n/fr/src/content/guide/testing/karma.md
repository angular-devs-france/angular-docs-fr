<!-- source: 272b58b39ca84ed6fbb6195fe9ee89086bb8dd0d -->
# Tester avec Karma et Jasmine

Si [Vitest](https://vitest.dev) est le test runner par défaut des nouveaux projets Angular, [Karma](https://karma-runner.github.io) reste un test runner pris en charge et largement utilisé. Ce guide explique comment tester votre application Angular avec le test runner Karma et le framework de test [Jasmine](https://jasmine.github.io).

## Mettre en place les outils Karma et Jasmine {#setting-up-karma-and-jasmine}

Vous pouvez configurer Karma et Jasmine pour un nouveau projet ou les ajouter à un projet existant.

### Pour les nouveaux projets {#for-new-projects}

Pour créer un nouveau projet avec Karma et Jasmine préconfigurés, exécutez la commande `ng new` avec l'option `--test-runner=karma` :

```shell
ng new my-karma-app --test-runner=karma
```

### Pour les projets existants {#for-existing-projects}

Pour ajouter Karma et Jasmine à un projet existant, suivez ces étapes :

1.  **Installez les packages nécessaires :**

    <docs-code-multifile>
      <docs-code header="npm" language="shell">
        npm install --save-dev karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core @types/jasmine
      </docs-code>
      <docs-code header="yarn" language="shell">
        yarn add --dev karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core @types/jasmine
      </docs-code>
      <docs-code header="pnpm" language="shell">
        pnpm add -D karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core @types/jasmine
      </docs-code>
      <docs-code header="bun" language="shell">
        bun add --dev karma karma-chrome-launcher karma-coverage karma-jasmine karma-jasmine-html-reporter jasmine-core @types/jasmine
      </docs-code>
    </docs-code-multifile>

2.  **Configurez le test runner dans `angular.json` :**

    Dans votre fichier `angular.json`, trouvez la cible `test` et définissez l'option `runner` à `karma` :

    ```json
    {
      // ...
      "projects": {
        "your-project-name": {
          // ...
          "architect": {
            "test": {
              "builder": "@angular/build:unit-test",
              "options": {
                "runner": "karma"
                // ... other options
              }
            }
          }
        }
      }
    }
    ```

3.  **Mettez à jour `tsconfig.spec.json` pour les types Jasmine :**

    Pour que TypeScript reconnaisse les fonctions de test globales comme `describe` et `it`, ajoutez `"jasmine"` au tableau `types` de votre `tsconfig.spec.json` :

    ```json
    {
      // ...
      "compilerOptions": {
        // ...
        "types": ["jasmine"]
      }
      // ...
    }
    ```

## Exécuter les tests {#running-tests}

Une fois votre projet configuré, exécutez les tests avec la commande [`ng test`](cli/test) :

```shell
ng test
```

La commande `ng test` compile l'application en _mode watch_ et lance le [test runner Karma](https://karma-runner.github.io).

La sortie de la console ressemble à ceci :

```shell

02 11 2022 09:08:28.605:INFO [karma-server]: Karma v6.4.1 server started at http://localhost:9876/
02 11 2022 09:08:28.607:INFO [launcher]: Launching browsers Chrome with concurrency unlimited
02 11 2022 09:08:28.620:INFO [launcher]: Starting browser Chrome
02 11 2022 09:08:31.312:INFO [Chrome]: Connected on socket -LaEYvD2R7MdcS0-AAAB with id 31534482
Chrome: Executed 3 of 3 SUCCESS (0.193 secs / 0.172 secs)
TOTAL: 3 SUCCESS

```

Le résultat des tests s'affiche dans le navigateur grâce à [Karma Jasmine HTML Reporter](https://github.com/dfederm/karma-jasmine-html-reporter).

<img alt="Jasmine HTML Reporter dans le navigateur" src="assets/images/guide/testing/initial-jasmine-html-reporter.png">

Cliquez sur une ligne de test pour relancer uniquement ce test, ou cliquez sur une description pour relancer les tests du groupe de tests sélectionné (la « suite de tests »).

Pendant ce temps, la commande `ng test` surveille les modifications. Pour le constater, apportez une petite modification à un fichier source et enregistrez. Les tests s'exécutent à nouveau, le navigateur se rafraîchit et les nouveaux résultats de test apparaissent.

## Configuration {#configuration}

L'Angular CLI se charge pour vous de la configuration de Jasmine et de Karma. Elle construit la configuration complète en mémoire, à partir des options indiquées dans le fichier `angular.json`.

### Personnaliser la configuration de Karma {#customizing-karma-configuration}

Si vous souhaitez personnaliser Karma, vous pouvez créer un fichier `karma.conf.js` en exécutant la commande suivante :

```shell
ng generate config karma
```

HELPFUL: Pour en savoir plus sur la configuration de Karma, consultez le [guide de configuration de Karma](https://karma-runner.github.io/6.4/config/configuration-file.html).

### Définir le test runner dans `angular.json` {#setting-the-test-runner-in-angularjson}

Pour définir explicitement Karma comme test runner de votre projet, repérez la cible `test` dans votre fichier `angular.json` et définissez l'option `runner` à `karma` :

```json
{
  // ...
  "projects": {
    "your-project-name": {
      // ...
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "runner": "karma"
            // ... other options
          }
        }
      }
    }
  }
}
```

## Imposer une couverture de code minimale {#code-coverage-enforcement}

Pour imposer un niveau minimal de couverture de code, vous pouvez utiliser la propriété `check` de la section `coverageReporter` de votre fichier `karma.conf.js`.

Par exemple, pour exiger une couverture minimale de 80 % :

```javascript
coverageReporter: {
  dir: require('path').join(__dirname, './coverage/<project-name>'),
  subdir: '.',
  reporters: [
    { type: 'html' },
    { type: 'text-summary' }
  ],
  check: {
    global: {
      statements: 80,
      branches: 80,
      functions: 80,
      lines: 80
    }
  }
}
```

L'exécution des tests échoue alors si les seuils de couverture indiqués ne sont pas atteints.

## Tester en intégration continue {#testing-in-continuous-integration}

Pour exécuter vos tests Karma dans un environnement de CI, utilisez la commande suivante :

```shell
ng test --no-watch --no-progress --browsers=ChromeHeadless
```

NOTE: Les options `--no-watch` et `--no-progress` sont indispensables pour Karma dans les environnements de CI, afin que les tests s'exécutent une seule fois et se terminent proprement. L'option `--browsers=ChromeHeadless` est aussi essentielle pour exécuter les tests dans un environnement de navigateur sans interface graphique.

## Déboguer les tests {#debugging-tests}

Si vos tests ne fonctionnent pas comme prévu, vous pouvez les inspecter et les déboguer dans le navigateur.

Pour déboguer une application avec le test runner Karma :

1.  Affichez la fenêtre de navigateur de Karma. Consultez [Configuration pour les tests](guide/testing#set-up-for-testing) si vous avez besoin d'aide pour cette étape.
2.  Cliquez sur le bouton **DEBUG** pour ouvrir un nouvel onglet du navigateur et relancer les tests.
3.  Ouvrez les **outils de développement** du navigateur. Sous Windows, appuyez sur `Ctrl-Shift-I`. Sous macOS, appuyez sur `Command-Option-I`.
4.  Choisissez la section **Sources**.
5.  Appuyez sur `Control/Command-P`, puis commencez à saisir le nom de votre fichier de test pour l'ouvrir.
6.  Placez un point d'arrêt dans le test.
7.  Rafraîchissez le navigateur et observez qu'il s'arrête au point d'arrêt.

<img alt="Débogage avec Karma" src="assets/images/guide/testing/karma-1st-spec-debug.png">
