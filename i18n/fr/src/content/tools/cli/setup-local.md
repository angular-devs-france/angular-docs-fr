<!-- source: ff71578f6ce6bc0667b7fd330dd2122bca478657 -->
# Configurer l'environnement local et le workspace

Ce guide explique comment configurer votre environnement pour le développement Angular à l'aide de l'[Angular CLI](cli 'Référence des commandes de la CLI').
Il explique comment installer la CLI, créer un workspace initial et une application de démarrage, puis exécuter cette application en local pour vérifier votre configuration.

<docs-callout title="Essayer Angular sans configuration locale">

Si vous découvrez Angular, vous pouvez commencer par [Essayez dès maintenant !](tutorials/learn-angular), qui présente les fondamentaux d'Angular dans votre navigateur.
Ce tutoriel autonome tire parti de l'environnement interactif [StackBlitz](https://stackblitz.com) pour le développement en ligne.
Vous n'avez pas besoin de configurer votre environnement local avant d'être prêt.

</docs-callout>

## Avant de commencer {#before-you-start}

Pour utiliser l'Angular CLI, vous devez être familiarisé avec les technologies suivantes :

<docs-pill-row>
  <docs-pill href="https://developer.mozilla.org/en-US/docs/Web/JavaScript" title="JavaScript"/>
  <docs-pill href="https://developer.mozilla.org/en-US/docs/Web/HTML" title="HTML"/>
  <docs-pill href="https://developer.mozilla.org/en-US/docs/Web/CSS" title="CSS"/>
</docs-pill-row>

Vous devez aussi savoir utiliser les outils en ligne de commande (CLI) et avoir une compréhension générale des shells de commande.
Une connaissance de [TypeScript](https://www.typescriptlang.org) est utile, mais pas obligatoire.

## Dépendances {#dependencies}

Pour installer l'Angular CLI sur votre système local, vous devez installer [Node.js](https://nodejs.org/).
L'Angular CLI utilise Node et son gestionnaire de packages associé, npm, pour installer et exécuter des outils JavaScript en dehors du navigateur.

[Téléchargez et installez Node.js](https://nodejs.org/en/download), qui inclut également la CLI `npm`.
Angular nécessite une version [LTS active ou LTS de maintenance](https://nodejs.org/en/about/previous-releases) de Node.js.
Consultez le guide de [compatibilité des versions d'Angular](reference/versions) pour plus d'informations.

## Installer l'Angular CLI {#install-the-angular-cli}

Pour installer l'Angular CLI, ouvrez une fenêtre de terminal et exécutez la commande suivante :

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

### Stratégie d'exécution PowerShell {#powershell-execution-policy}

Sur les postes clients Windows, l'exécution des scripts PowerShell est désactivée par défaut : la commande ci-dessus peut donc échouer avec une erreur.
Pour autoriser l'exécution des scripts PowerShell, nécessaire pour les binaires globaux npm, vous devez définir la <a href="https://docs.microsoft.com/powershell/module/microsoft.powershell.core/about/about_execution_policies" target="_blank">stratégie d'exécution</a> suivante :

```sh

Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned

```

Lisez attentivement le message affiché après l'exécution de la commande et suivez les instructions. Assurez-vous de bien comprendre les implications de la définition d'une stratégie d'exécution.

### Permissions Unix {#unix-permissions}

Sur certaines configurations de type Unix, les scripts globaux peuvent appartenir à l'utilisateur root : la commande ci-dessus peut donc échouer avec une erreur de permission.
Exécutez-la avec `sudo` pour lancer la commande en tant qu'utilisateur root, et saisissez votre mot de passe lorsque vous y êtes invité :

<docs-code-multifile>
   <docs-code
     header="npm"
     language="shell"
     >
     sudo npm install -g @angular/cli
     </docs-code>
   <docs-code
     header="pnpm"
     language="shell"
     >
     sudo pnpm install -g @angular/cli
     </docs-code>
   <docs-code
     header="yarn"
     language="shell"
     >
     sudo yarn global add @angular/cli
     </docs-code>
   <docs-code
     header="bun"
     language="shell"
     >
     sudo bun install -g @angular/cli
     </docs-code>

 </docs-code-multifile>

Assurez-vous de bien comprendre les implications de l'exécution de commandes en tant que root.

## Créer un workspace et une application initiale {#create-a-workspace-and-initial-application}

Vous développez des applications dans le contexte d'un **workspace** Angular.

Pour créer un nouveau workspace et une application de démarrage initiale, exécutez la commande CLI `ng new` en indiquant le nom `my-app`, comme ci-dessous, puis répondez aux questions sur les fonctionnalités à inclure :

```shell

ng new my-app

```

L'Angular CLI installe chaque package npm Angular nécessaire ainsi que les autres dépendances.
Cela peut prendre quelques minutes.

La CLI crée un nouveau workspace et une petite application de bienvenue, prête à être exécutée, dans un nouveau répertoire portant le même nom que le workspace.
Placez-vous dans ce nouveau répertoire pour que les commandes suivantes utilisent ce workspace.

```shell

cd my-app

```

## Exécuter l'application {#run-the-application}

L'Angular CLI inclut un serveur de développement qui vous permet de compiler et de servir votre application en local. Exécutez la commande suivante :

```shell

ng serve --open

```

La commande `ng serve` lance le serveur et surveille vos fichiers : elle recompile l'application et recharge le navigateur à mesure que vous modifiez ces fichiers.

L'option `--open` (ou simplement `-o`) ouvre automatiquement votre navigateur sur `http://localhost:4200/` pour afficher l'application générée.

## Workspaces et fichiers de projet {#workspaces-and-project-files}

La commande [`ng new`](cli/new) crée un dossier de [workspace Angular](reference/configs/workspace-config) et génère une nouvelle application à l'intérieur.
Un workspace peut contenir plusieurs applications et libraries.
L'application initiale créée par la commande [`ng new`](cli/new) se trouve dans le répertoire racine du workspace.
Lorsque vous générez une application ou une library supplémentaire dans un workspace existant, elle est placée par défaut dans un sous-dossier `projects/`.

Une application nouvellement générée contient les fichiers source d'un composant racine et de son template.
Chaque application possède un dossier `src` qui contient ses composants, ses données et ses assets.

Vous pouvez modifier directement les fichiers générés, ou en ajouter et les modifier à l'aide des commandes de la CLI.
Utilisez la commande [`ng generate`](cli/generate) pour ajouter de nouveaux fichiers pour des composants, directives, pipes, services supplémentaires, et plus encore.
Les commandes comme [`ng add`](cli/add) et [`ng generate`](cli/generate), qui créent des applications et des libraries ou agissent sur elles, doivent être exécutées
depuis un workspace. À l'inverse, les commandes comme `ng new` doivent être exécutées _en dehors_ d'un workspace, car elles en créent un nouveau.

## Étapes suivantes {#next-steps}

- Découvrez la [structure des fichiers](reference/configs/file-structure) et la [configuration](reference/configs/workspace-config) du workspace généré.

- Testez votre nouvelle application avec [`ng test`](cli/test).

- Générez du code de base comme des composants, des directives et des pipes avec [`ng generate`](cli/generate).

- Déployez votre nouvelle application et rendez-la accessible à de vrais utilisateurs avec [`ng deploy`](cli/deploy).

- Configurez et exécutez des tests de bout en bout de votre application avec [`ng e2e`](cli/e2e).
