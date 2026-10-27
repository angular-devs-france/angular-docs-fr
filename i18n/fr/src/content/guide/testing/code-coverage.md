<!-- source: d50ed8ebb7d39fbf4b72b844795bfd55898db01e -->
# Couverture de code

Les rapports de couverture de code vous montrent les parties de votre base de code qui ne sont peut-être pas correctement testées par vos tests unitaires.

## Prérequis {#prerequisites}

Pour générer des rapports de couverture de code avec Vitest, vous devez installer le package `@vitest/coverage-v8` :

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install --save-dev @vitest/coverage-v8
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add --dev @vitest/coverage-v8
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add -D @vitest/coverage-v8
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add --dev @vitest/coverage-v8
  </docs-code>
</docs-code-multifile>

## Générer un rapport {#generating-a-report}

Pour générer un rapport de couverture, ajoutez l'option `--coverage` à la commande `ng test` :

```shell
ng test --coverage
```

Une fois les tests exécutés, la commande crée un nouveau répertoire `coverage/` dans le projet. Ouvrez le fichier `index.html` pour afficher un rapport contenant votre code source et ses valeurs de couverture de code.

Si vous voulez créer des rapports de couverture de code à chaque exécution des tests, vous pouvez définir l'option `coverage` à `true` dans votre fichier `angular.json` :

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "coverage": true
          }
        }
      }
    }
  }
}
```

## Imposer des seuils de couverture de code {#enforcing-code-coverage-thresholds}

Les pourcentages de couverture de code vous permettent d'estimer quelle part de votre code est testée. Si votre équipe fixe un minimum à couvrir par des tests unitaires, vous pouvez imposer ce minimum dans votre configuration.

Par exemple, supposons que vous vouliez que la base de code ait une couverture de code d'au moins 80 %. Pour cela, ajoutez l'option `coverageThresholds` à votre fichier `angular.json` :

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "coverage": true,
            "coverageThresholds": {
              "statements": 80,
              "branches": 80,
              "functions": 80,
              "lines": 80
            }
          }
        }
      }
    }
  }
}
```

Désormais, si votre couverture descend sous 80 % lorsque vous exécutez vos tests, la commande échoue.

## Configuration avancée {#advanced-configuration}

Vous pouvez configurer plusieurs autres options de couverture dans votre fichier `angular.json` :

- `coverageInclude` : patterns glob des fichiers à inclure dans le rapport de couverture.
- `coverageExclude` : patterns glob des fichiers à exclure du rapport de couverture.
- `coverageReporters` : un tableau de reporters à utiliser (par exemple `html`, `lcov`, `json`).
- `coverageWatermarks` : un objet qui définit les seuils `[low, high]` du reporter HTML, qui peuvent influer sur le code couleur du rapport.

```json
{
  "projects": {
    "your-project-name": {
      "architect": {
        "test": {
          "builder": "@angular/build:unit-test",
          "options": {
            "coverage": true,
            "coverageReporters": ["html", "lcov"],
            "coverageWatermarks": {
              "statements": [50, 80],
              "branches": [50, 80],
              "functions": [50, 80],
              "lines": [50, 80]
            }
          }
        }
      }
    }
  }
}
```
