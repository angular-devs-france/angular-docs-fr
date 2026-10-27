<!-- source: 824e2e8ca7ede77658066e40dab2da4f7eef4f7f -->
<docs-decorative-header title="Installation" imgSrc="adev/src/assets/images/what_is_angular.svg"> <!-- markdownlint-disable-line -->
</docs-decorative-header>

Démarrez rapidement avec Angular grâce aux projets en ligne, ou en local avec votre terminal.

## Essayer en ligne {#play-online}

Si vous voulez simplement essayer Angular dans votre navigateur sans configurer de projet, vous pouvez utiliser notre bac à sable en ligne :

<docs-card title="Playground" href="/playground" link="Ouvrir dans le Playground" iconImgSrc="adev/src/assets/icons/playground.svg" titleInline>
Le moyen le plus rapide d'essayer une application Angular. Aucune configuration nécessaire.
</docs-card>

## Créer un nouveau projet en local {#set-up-a-new-project-locally}

Si vous démarrez un nouveau projet, vous voudrez très probablement créer un projet local afin de pouvoir utiliser des outils comme Git.

### Prérequis {#prerequisites}

- **Node.js** - [v22.22.3 ou plus récent](/reference/versions)
- **Éditeur de texte** - Nous recommandons [Visual Studio Code](https://code.visualstudio.com/)
- **Terminal** - Nécessaire pour exécuter les commandes de l'[Angular CLI](/tools/cli)
- **Outil de développement** - Pour améliorer votre flux de développement, nous recommandons l'[Angular Language Service](/tools/language-service)

### Instructions {#instructions}

Le guide suivant vous accompagne dans la création d'un projet Angular en local.

#### Installer l'Angular CLI {#install-angular-cli}

Ouvrez un terminal (si vous utilisez [Visual Studio Code](https://code.visualstudio.com/), vous pouvez ouvrir un [terminal intégré](https://code.visualstudio.com/docs/editor/integrated-terminal)) et exécutez la commande suivante :

<docs-code-multifile>
  <docs-code
    header="npm"
    language="shell"
    >
    npm install -g @angular/cli
    </docs-code>
  <docs-code
    header="pnpm"
    language="shell"
    >
    pnpm install -g @angular/cli
    </docs-code>
  <docs-code
    header="yarn"
    language="shell"
    >
    yarn global add @angular/cli
    </docs-code>
  <docs-code
    header="bun"
    language="shell"
    >
    bun install -g @angular/cli
    </docs-code>
</docs-code-multifile>

Si vous rencontrez des problèmes pour exécuter cette commande sous Windows ou Unix, consultez la [documentation de la CLI](/tools/cli/setup-local#install-the-angular-cli) pour plus d'informations.

#### Créer un nouveau projet {#create-a-new-project}

Dans votre terminal, exécutez la commande CLI [`ng new`](cli/new) avec le nom de projet souhaité. Dans les exemples suivants, nous utiliserons le nom de projet `my-first-angular-app`.

```shell
ng new <project-name>
```

Plusieurs options de configuration vous seront proposées pour votre projet. Utilisez les flèches et la touche Entrée pour naviguer et sélectionner les options souhaitées.

Si vous n'avez pas de préférence, appuyez simplement sur Entrée pour accepter les options par défaut et poursuivre la configuration.

Une fois les options de configuration choisies et la configuration terminée par la CLI, vous devriez voir le message suivant :

```text
✔ Packages installed successfully.
    Successfully initialized git.
```

Vous êtes maintenant prêt à exécuter votre projet en local !

#### Exécuter votre nouveau projet en local {#running-your-new-project-locally}

Dans votre terminal, placez-vous dans votre nouveau projet Angular.

```shell
cd my-first-angular-app
```

Toutes vos dépendances devraient être installées à ce stade (vous pouvez le vérifier en contrôlant la présence d'un dossier `node_modules` dans votre projet). Vous pouvez donc démarrer votre projet avec la commande :

```shell
npm start
```

Si tout se passe bien, vous devriez voir un message de confirmation similaire dans votre terminal :

```text
Watch mode enabled. Watching for file changes...
NOTE: Raw file sizes do not reflect development server per-request transformations.
  ➜  Local:   http://localhost:4200/
  ➜  press h + enter to show help
```

Vous pouvez maintenant ouvrir l'adresse indiquée dans `Local` (par exemple `http://localhost:4200`) pour voir votre application. Bon code ! 🎉

### Utiliser l'IA pour le développement {#using-ai-for-development}

Pour commencer à développer dans votre IDE assisté par IA préféré, [consultez les règles de prompt et les bonnes pratiques Angular](/ai/develop-with-ai).

## Prochaines étapes {#next-steps}

Maintenant que vous avez créé votre projet Angular, vous pouvez en apprendre davantage sur Angular avec notre [guide des Fondamentaux](/essentials) ou choisir un sujet dans nos guides approfondis !
