<!-- source: 2d96fd5f2af210b20e6bef522a4ca87188238179 -->
# Serveur MCP de l'Angular CLI

L'Angular CLI inclut un serveur Model Context Protocol (MCP) qui permet aux assistants IA (comme Cursor, Antigravity, JetBrains AI, etc.) d'interagir directement avec l'Angular CLI. Il fournit des outils pour la génération de code, l'analyse du workspace et l'exécution de builds et de tests.

<docs-callout title="Intégration avec les Agent Skills Angular pour l'IA">
  Si votre environnement host prend en charge des Agent Skills personnalisées (comme Antigravity), vous pouvez combiner le serveur MCP de l'Angular CLI avec les [Angular AI Skills](https://angular.dev/ai/agent-skills) officielles. Tandis que les skills fournissent à l'agent des recommandations approfondies sous forme d'instructions et des standards de code, le serveur MCP fournit les outils d'action (comme compiler, lancer les tests et analyser les workspaces) pour appliquer ces recommandations, ce qui donne un agent de développement complet et puissant.
</docs-callout>

## Premiers pas {#get-started}

Pour utiliser le serveur MCP, configurez votre environnement host (IDE ou CLI) pour qu'il exécute `npx @angular/cli mcp`.

<docs-tab-group>
  <docs-tab label="Antigravity IDE">
    Créez un fichier nommé `.antigravity/mcp.json` à la racine de votre projet :

    ```json
    {
      "mcpServers": {
        "angular-cli": {
          "command": "npx",
          "args": ["-y", "@angular/cli", "mcp"]
        }
      }
    }
    ```

  </docs-tab>

  <docs-tab label="Cursor">
    Créez `.cursor/mcp.json` à la racine du projet (ou globalement dans `~/.cursor/mcp.json`) :

    ```json
    {
      "mcpServers": {
        "angular-cli": {
          "command": "npx",
          "args": ["-y", "@angular/cli", "mcp"]
        }
      }
    }
    ```

  </docs-tab>

  <docs-tab label="VS Code">
    Créez `.vscode/mcp.json` :

    ```json
    {
      "servers": {
        "angular-cli": {
          "command": "npx",
          "args": ["-y", "@angular/cli", "mcp"]
        }
      }
    }
    ```

  </docs-tab>
</docs-tab-group>

## Outils disponibles (par défaut) {#available-tools-default}

Lorsque le serveur MCP est activé, les agents IA ont accès aux outils suivants :

| Nom                        | Description                                                                                               |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------- |
| `ai_tutor`                  | Lance un tuteur Angular interactif propulsé par l'IA.                                                         |
| `devserver_start`           | Démarre de manière asynchrone un serveur de développement (`ng serve`). Rend la main immédiatement.                                     |
| `devserver_stop`            | Arrête le serveur de développement.                                                                                     |
| `devserver_wait_for_build`  | Renvoie les logs du build le plus récent d'un serveur de développement en cours d'exécution.                                        |
| `get_best_practices`        | Récupère le guide des bonnes pratiques Angular (essentiel pour les composants standalone, les formulaires typés, etc.).        |
| `list_projects`             | Liste toutes les applications et libraries du workspace en lisant `angular.json`.                          |
| `onpush_zoneless_migration` | Analyse le code et fournit un plan pour le migrer vers la détection de changements `OnPush` (prérequis pour le mode zoneless). |
| `run_target`                | Exécute une cible configurée (par ex. build, test, lint, e2e, deploy).                                      |
| `search_documentation`      | Effectue une recherche dans la documentation officielle sur `https://angular.dev`.                                             |

## Workflows courants {#common-workflows}

Ces workflows montrent comment les assistants IA coordonnent différents outils MCP pour réaliser automatiquement des scénarios de développement complexes.

### 1. Optimisation des performances : migration zoneless et OnPush {#1-performance-tuning-zoneless--onpush-migration}

L'agent IA optimise les performances de la détection de changements et migre les composants vers un état compatible avec le mode zoneless.

1. **Découvrir le workspace** : l'agent IA appelle `list_projects` pour localiser les composants, les projets et les configurations de styles et de tests du workspace.
2. **Modernisation par schematic (prérequis)** : l'agent IA exécute les migrations vers les signals nécessaires au préalable à l'aide des commandes standard `ng generate` (par ex. Signal Inputs, Signal Queries).
3. **Planifier la migration** : l'agent IA appelle `onpush_zoneless_migration` avec le chemin absolu du répertoire ou du fichier du composant.
4. **Appliquer les changements** : l'agent IA applique automatiquement à la base de code l'unique changement actionnable renvoyé par l'outil.
5. **Vérifier les changements** : l'agent IA lance les tests unitaires en appelant `run_target` avec le paramètre target défini à `"test"`.
6. **Répéter** : l'agent IA appelle de nouveau `onpush_zoneless_migration` pour obtenir l'étape suivante, et recommence jusqu'à ce que l'outil indique que la migration est terminée.

### 2. Développement de fonctionnalités et boucle TDD {#2-feature-development--tdd-loop}

L'agent IA automatise la recherche, l'implémentation et la vérification lors du développement de nouvelles fonctionnalités.

1. **Recherche sur les API et la syntaxe** : l'agent IA utilise `search_documentation` pour consulter les API Angular ou les règles de syntaxe (par ex. les options du bloc `@defer`).
2. **Charger les standards de code** : l'agent IA appelle `get_best_practices` avec le chemin du workspace pour charger les règles de code alignées sur la version d'Angular.
3. **Démarrer le serveur de développement local** : l'agent IA démarre un serveur en arrière-plan en appelant `devserver_start`.
4. **Surveiller le build** : l'agent IA utilise `devserver_wait_for_build` pour suivre les logs de build et s'assurer que la compilation réussit pendant qu'il modifie le code.
5. **Écrire et exécuter les tests** : l'agent IA identifie le framework de test du projet (par ex. Jasmine, Jest, Vitest) via `list_projects`, écrit le fichier de test correspondant et lance les tests avec `run_target` et `"test"`.
6. **Arrêter le serveur de développement** : une fois terminé, l'agent IA arrête le serveur de développement actif en appelant `devserver_stop`.

### 3. Intégration et apprentissage des développeurs {#3-developer-onboarding-and-learning}

L'agent IA guide le développeur à travers les concepts d'Angular dans un bac à sable interactif.

1. **Découvrir les projets** : l'agent IA appelle `list_projects` pour parcourir le workspace et identifier la structure de la base de code.
2. **Lancer le tuteur** : l'agent IA exécute `ai_tutor` pour charger les instructions du programme, le persona et les règles de tutorat.
3. **Suivre le programme** : l'agent IA guide l'utilisateur tout au long du programme, en expliquant les concepts et en lui indiquant quels composants construire ou modifier.
4. **Implémenter et vérifier** : l'agent IA aide à implémenter le code du bac à sable et vérifie les changements avec `run_target` et `"test"` ou `"build"`.

## Options de la commande {#command-options}

Vous pouvez passer des arguments au serveur MCP dans le tableau `args` de votre configuration :

- `--read-only` : n'enregistre que les outils qui ne modifient pas le projet.
- `--local-only` : n'enregistre que les outils qui ne nécessitent pas de connexion internet.

Exemple pour le mode lecture seule :

```json
"args": ["-y", "@angular/cli", "mcp", "--read-only"]
```
