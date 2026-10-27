<!-- source: 13090475d0ad56ad0c5b432c960835e3e397de4f -->
# L'Angular CLI

L'Angular CLI est un outil en ligne de commande qui vous permet de générer la structure, de développer, de tester, de déployer et de maintenir des applications Angular directement depuis un terminal.

L'Angular CLI est publiée sur npm sous la forme du package `@angular/cli` et inclut un binaire nommé `ng`. Les commandes qui invoquent `ng` utilisent l'Angular CLI.

<docs-callout title="Essayer Angular sans installation locale">

Si vous découvrez Angular, vous pouvez commencer par [Essayez maintenant !](tutorials/learn-angular), qui présente les fondamentaux d'Angular dans le contexte d'une application de boutique en ligne basique, prête à l'emploi, que vous pouvez examiner et modifier.
Ce tutoriel autonome tire parti de l'environnement interactif [StackBlitz](https://stackblitz.com) pour le développement en ligne.
Vous n'avez pas besoin de configurer votre environnement local avant d'être prêt.

</docs-callout>

<docs-card-container>
  <docs-card title="Premiers pas" link="Commencer" href="tools/cli/setup-local">
    Installez l'Angular CLI pour créer et compiler votre première application.
  </docs-card>
  <docs-card title="Référence des commandes" link="En savoir plus" href="cli">
    Découvrez les commandes de la CLI qui vous rendront plus productif avec Angular.
  </docs-card>
  <docs-card title="Schematics" link="En savoir plus" href="tools/cli/schematics">
    Créez et exécutez des schematics pour générer et modifier automatiquement les fichiers source de votre application.
  </docs-card>
  <docs-card title="Builders" link="En savoir plus" href="tools/cli/cli-builder">
    Créez et exécutez des builders pour effectuer des transformations complexes de votre code source vers les sorties de build générées.
  </docs-card>
</docs-card-container>

## Syntaxe du langage de commande de la CLI {#cli-command-language-syntax}

L'Angular CLI suit globalement les conventions Unix/POSIX pour la syntaxe des options.

### Options booléennes {#boolean-options}

Les options booléennes ont deux formes : `--this-option` définit l'option à `true`, `--no-this-option` la définit à `false`.
Vous pouvez aussi utiliser `--this-option=false` ou `--this-option=true`.
Si aucune des deux n'est fournie, l'option conserve son état par défaut, tel qu'indiqué dans la documentation de référence.

### Options de type tableau {#array-options}

Les options de type tableau peuvent être fournies sous deux formes : `--option value1 value2` ou `--option value1 --option value2`.

### Options clé/valeur {#key-value-options}

Certaines options comme `--define` attendent comme valeurs un tableau de paires `key=value`.
Comme les options de type tableau, les options clé/valeur peuvent être fournies sous deux formes :
`--define 'KEY_1="value1"' KEY_2=true` ou `--define 'KEY_1="value1"' --define KEY_2=true`.

### Chemins relatifs {#relative-paths}

Les options qui désignent des fichiers peuvent recevoir des chemins absolus, ou des chemins relatifs au répertoire de travail courant, qui est généralement la racine du workspace ou du projet.
