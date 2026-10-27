<!-- source: da915ba7d09c1fe645ba1c8d2a04764f5e7bdf15 -->
# Tests de bout en bout

Les tests de bout en bout (E2E) sont une forme de test qui permet de vérifier que l'ensemble de votre application fonctionne comme prévu du début à la fin, c'est-à-dire _« de bout en bout »_. Les tests E2E diffèrent des tests unitaires en ce qu'ils sont complètement découplés des détails d'implémentation sous-jacents de votre code. Ils servent généralement à valider une application d'une manière qui imite la façon dont un utilisateur interagirait avec elle. Cette page est un guide pour démarrer avec les tests de bout en bout dans Angular à l'aide de l'Angular CLI.

## Mettre en place les tests E2E {#set-up-e2e-testing}

L'Angular CLI télécharge et installe tout ce dont vous avez besoin pour exécuter les tests de bout en bout de votre application Angular.

```shell

ng e2e

```

La commande `ng e2e` vérifie d'abord si votre projet possède la cible « e2e ». Si elle ne la trouve pas, la CLI vous demande quel package e2e vous souhaitez utiliser et vous guide dans sa mise en place.

```text

Cannot find "e2e" target for the specified project.
You can add a package that implements these capabilities.

For example:
Playwright: ng add playwright-ng-schematics
Cypress: ng add @cypress/schematic
Nightwatch: ng add @nightwatch/schematics
WebdriverIO: ng add @wdio/schematics
Puppeteer: ng add @puppeteer/ng-schematics

Would you like to add a package with "e2e" capabilities now?
❯ No
Playwright
Cypress
Nightwatch
WebdriverIO
Puppeteer

```

Si le test runner que vous souhaitez utiliser ne figure pas dans la liste ci-dessus, vous pouvez ajouter manuellement un package avec `ng add`.

## Exécuter les tests E2E {#running-e2e-tests}

Maintenant que votre application est configurée pour les tests de bout en bout, vous pouvez exécuter la même commande pour lancer vos tests.

```shell

ng e2e

```

Notez qu'il n'y a rien de « spécial » à exécuter vos tests avec l'un des packages e2e intégrés. La commande `ng e2e` ne fait en réalité qu'exécuter le builder `e2e` en arrière-plan. Vous pouvez toujours [créer votre propre builder personnalisé](tools/cli/cli-builder#creating-a-builder) nommé `e2e` et l'exécuter avec `ng e2e`.

## Plus d'informations sur les outils de test de bout en bout {#more-information-on-end-to-end-testing-tools}

| Outil de test | Détails                                                                                                              |
| :----------- | :------------------------------------------------------------------------------------------------------------------- |
| Cypress      | [Premiers pas avec Cypress](https://docs.cypress.io/guides/end-to-end-testing/writing-your-first-end-to-end-test) |
| Nightwatch   | [Premiers pas avec Nightwatch](https://nightwatchjs.org/guide/writing-tests/introduction.html)                    |
| WebdriverIO  | [Premiers pas avec Webdriver.io](https://webdriver.io/docs/gettingstarted)                                        |
| Playwright   | [Premiers pas avec Playwright](https://playwright.dev/docs/writing-tests)                                         |
| Puppeteer    | [Premiers pas avec Puppeteer](https://pptr.dev)                                                                   |
