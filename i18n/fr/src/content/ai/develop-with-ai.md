<!-- source: 3d402700b8957ae9a599b07df3873c3ff9bd4efe -->
# Prompts pour LLM et configuration des IDE IA

Générer du code avec des grands modèles de langage (LLM) suscite un intérêt croissant chez les développeurs. Si les LLM sont souvent capables de générer du code fonctionnel, il peut être difficile de générer du code pour des frameworks en évolution constante comme Angular.

Les instructions avancées et le prompting deviennent un standard pour accompagner la génération de code moderne avec des détails propres au domaine. Cette section contient du contenu et des ressources sélectionnés pour favoriser une génération de code plus précise pour Angular avec les LLM.

## Prompts personnalisés et instructions système {#custom-prompts-and-system-instructions}

Améliorez votre expérience de génération de code avec les LLM en utilisant l'un des fichiers personnalisés suivants, propres au domaine.

NOTE: Ces fichiers seront mis à jour régulièrement pour rester alignés sur les conventions d'Angular.

Voici un ensemble d'instructions pour aider les LLM à générer du code correct qui suit les bonnes pratiques Angular. Ce fichier peut être ajouté comme instructions système à votre outillage IA ou inclus avec votre prompt comme contexte.

<docs-code language="md" path="packages/core/resources/best-practices.md" class="compact"/>

<a download href="/assets/context/best-practices.md" target="_blank">Cliquez ici pour télécharger le fichier best-practices.md.</a>

## Fichiers de règles {#rules-files}

Plusieurs outils, comme <a href="https://antigravity.google?utm_source=adev&utm_medium=website&utm_campaign=BUILD_WITH_AI_ANGULAR&utm_term=angular_devrel&utm_content=build_with_ai_angular_antigravity" target="_blank">Antigravity</a>, disposent de fichiers de règles utiles pour fournir un contexte essentiel aux LLM.

| Environnement/IDE      | Fichier de règles                                                                                                             | Instructions d'installation                                                                                                                                       |
| :------------------- | :--------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Antigravity          | <a download href="/assets/context/GEMINI.md" target="_blank">GEMINI.md</a>                                             | <a href="https://antigravity.google/docs/rules-workflows" target="_blank">Configurer `GEMINI.md`</a>                                                             |
| IDE basés sur Copilot | <a download="copilot-instructions.md" href="/assets/context/guidelines.md" target="_blank">copilot-instructions.md</a> | <a href="https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions" target="_blank">Configurer `.github/copilot-instructions.md`</a> |
| Cursor               | <a download href="/assets/context/angular-20.mdc" target="_blank">cursor.md</a>                                        | <a href="https://docs.cursor.com/context/rules" target="_blank">Configurer `cursorrules.md`</a>                                                                  |
| IDE JetBrains       | <a download href="/assets/context/AGENTS.md" target="_blank">AGENTS.md</a>                                             | <a href="https://junie.jetbrains.com/docs/guidelines-and-memory.html#how-junie-cli-discovers-guidelines" target="_blank">Configurer `AGENTS.md`</a>              |
| VS Code              | <a download=".instructions.md" href="/assets/context/guidelines.md" target="_blank">.instructions.md</a>               | <a href="https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions" target="_blank">Configurer `.instructions.md`</a>                |
| Windsurf             | <a download href="/assets/context/guidelines.md" target="_blank">guidelines.md</a>                                     | <a href="https://docs.windsurf.com/windsurf/cascade/memories#rules" target="_blank">Configurer `guidelines.md`</a>                                               |

## Configurer le serveur MCP de l'Angular CLI {#angular-cli-mcp-server-setup}

L'Angular CLI inclut un [serveur Model Context Protocol (MCP)](https://modelcontextprotocol.io/) expérimental qui permet aux assistants IA de votre environnement de développement d'interagir avec l'Angular CLI.

[**Découvrez comment configurer le serveur MCP de l'Angular CLI**](/ai/mcp)

## Fournir du contexte avec `llms.txt` {#providing-context-with-llmstxt}

`llms.txt` est un standard proposé pour les sites web, conçu pour aider les LLM à mieux comprendre et traiter leur contenu. L'équipe Angular a développé deux versions de ce fichier pour aider les LLM, et les outils qui utilisent des LLM pour la génération de code, à produire un meilleur code Angular moderne.

- <a href="/llms.txt" target="_blank">llms.txt</a> - un fichier d'index qui fournit des liens vers les fichiers et ressources clés.
- <a href="/assets/context/llms-full.txt" target="_blank">llms-full.txt</a> - un ensemble de ressources compilé plus complet, qui décrit le fonctionnement d'Angular et la façon de construire des applications Angular.

Consultez aussi la [page de présentation](/ai) pour en savoir plus sur l'intégration de l'IA dans vos applications Angular.
