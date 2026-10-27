<!-- source: b12fce2a630cdcac832e440d04a208f3fcd231b5 -->
# Générer du code avec les schematics

Un schematic est un générateur de code à base de template qui prend en charge une logique complexe.
C'est un ensemble d'instructions pour transformer un projet logiciel en générant ou en modifiant du code.
Les schematics sont regroupés dans des collections et installés avec npm.

Une collection de schematics peut être un outil puissant pour créer, modifier et maintenir n'importe quel projet logiciel, mais elle est particulièrement utile pour adapter les projets Angular aux besoins spécifiques de votre organisation.
Vous pouvez par exemple utiliser des schematics pour générer des patterns d'interface courants ou des composants spécifiques, à partir de templates ou de mises en page prédéfinis.
Utilisez des schematics pour faire respecter des règles et des conventions d'architecture, afin de rendre vos projets cohérents et interopérables.

## Les schematics pour l'Angular CLI {#schematics-for-the-angular-cli}

Les schematics font partie de l'écosystème Angular.
L'Angular CLI utilise des schematics pour appliquer des transformations à un projet d'application web.
Vous pouvez modifier ces schematics et en définir de nouveaux, par exemple pour mettre à jour votre code afin de corriger les changements cassants d'une dépendance, ou pour ajouter une nouvelle option de configuration ou un nouveau framework à un projet existant.

Les schematics inclus dans la collection `@schematics/angular` sont exécutés par défaut par les commandes `ng generate` et `ng add`.
Le package contient des schematics nommés qui configurent les options disponibles dans la CLI pour les sous-commandes de `ng generate`, comme `ng generate component` et `ng generate service`.
Les sous-commandes de `ng generate` sont des raccourcis vers le schematic correspondant.
Pour indiquer et générer un schematic particulier, ou une collection de schematics, utilisez la forme longue :

```shell

ng generate my-schematic-collection:my-schematic-name

```

### Configurer les schematics de la CLI {#configuring-cli-schematics}

Un schéma JSON associé à un schematic indique à l'Angular CLI quelles options sont disponibles pour les commandes et sous-commandes, et en détermine les valeurs par défaut.
Ces valeurs par défaut peuvent être remplacées en fournissant une autre valeur pour une option sur la ligne de commande.
Consultez [Configuration du workspace](reference/configs/workspace-config) pour savoir comment modifier les valeurs par défaut des options de génération de votre workspace.

Les schémas JSON des schematics par défaut que la CLI utilise pour générer des projets et des parties de projets sont regroupés dans le package [`@schematics/angular`](https://github.com/angular/angular-cli/tree/main/packages/schematics/angular).
Le schéma décrit les options disponibles dans la CLI pour chacune des sous-commandes de `ng generate`, telles qu'elles apparaissent dans la sortie de `--help`.

## Développer des schematics pour les libraries {#developing-schematics-for-libraries}

En tant que développeur de library, vous pouvez créer vos propres collections de schematics personnalisés pour intégrer votre library à l'Angular CLI.

- Un _schematic d'ajout_ permet aux développeurs d'installer votre library dans un workspace Angular avec `ng add`
- Les _schematics de génération_ peuvent indiquer aux sous-commandes de `ng generate` comment modifier les projets, ajouter des configurations et des scripts, et générer la structure des artefacts définis dans votre library
- Un _schematic de mise à jour_ peut indiquer à la commande `ng update` comment mettre à jour les dépendances de votre library et s'adapter aux changements cassants lorsque vous publiez une nouvelle version

Pour plus de détails sur leur forme et sur la manière de les créer, consultez :

<docs-pill-row>
  <docs-pill href="tools/cli/schematics-authoring" title="Écrire des schematics"/>
  <docs-pill href="tools/cli/schematics-for-libraries" title="Schematics pour les libraries"/>
</docs-pill-row>

### Schematics d'ajout {#add-schematics}

Un _schematic d'ajout_ est généralement fourni avec une library, afin que celle-ci puisse être ajoutée à un projet existant avec `ng add`.
La commande `add` utilise votre gestionnaire de paquets pour télécharger les nouvelles dépendances, puis invoque un script d'installation implémenté sous forme de schematic.

Par exemple, le schematic [`@angular/material`](https://material.angular.dev/guide/schematics) indique à la commande `add` d'installer et de configurer Angular Material et ses thèmes, et d'enregistrer de nouveaux composants de démarrage qui peuvent être créés avec `ng generate`.
Inspirez-vous de celui-ci comme exemple et modèle pour votre propre schematic d'ajout.

Les libraries de partenaires et de tiers prennent aussi en charge l'Angular CLI avec des schematics d'ajout.
Par exemple, `@ng-bootstrap/schematics` ajoute [ng-bootstrap](https://ng-bootstrap.github.io) à une application, et `@clr/angular` installe et configure [Clarity de VMWare](https://clarity.design/documentation/get-started).

Un _schematic d'ajout_ peut aussi mettre à jour un projet avec des changements de configuration, ajouter des dépendances supplémentaires \(comme des polyfills\) ou générer du code d'initialisation propre au package.
Par exemple, le schematic `@angular/pwa` transforme votre application en PWA en ajoutant un manifeste d'application et un service worker.

### Schematics de génération {#generation-schematics}

Les schematics de génération sont des instructions pour la commande `ng generate`.
Les sous-commandes documentées utilisent les schematics de génération Angular par défaut, mais vous pouvez indiquer un autre schematic \(à la place d'une sous-commande\) pour générer un artefact défini dans votre library.

Angular Material, par exemple, fournit des schematics de génération pour les composants d'interface qu'il définit.
La commande suivante utilise l'un de ces schematics pour générer un `<mat-table>` Angular Material préconfiguré avec une source de données pour le tri et la pagination.

```shell

ng generate @angular/material:table <component-name>

```

### Schematics de mise à jour {#update-schematics}

La commande `ng update` permet de mettre à jour les dépendances de libraries de votre workspace.
Si vous ne fournissez aucune option ou utilisez l'option d'aide, la commande examine votre workspace et suggère les libraries à mettre à jour.

```shell

ng update
We analyzed your package.json, there are some packages to update:

    Name                                      Version                     Command to update
    --------------------------------------------------------------------------------
    @angular/cdk                       7.2.2 -> 7.3.1           ng update @angular/cdk
    @angular/cli                       7.2.3 -> 7.3.0           ng update @angular/cli
    @angular/core                      7.2.2 -> 7.2.3           ng update @angular/core
    @angular/material                  7.2.2 -> 7.3.1           ng update @angular/material
    rxjs                                      6.3.3 -> 6.4.0           ng update rxjs

```

Si vous passez à la commande un ensemble de libraries à mettre à jour, elle met à jour ces libraries, leurs dépendances peer, ainsi que les dépendances peer qui en dépendent.

HELPFUL: En cas d'incohérences \(par exemple, si des dépendances peer ne peuvent pas être satisfaites par une simple plage [semver](https://semver.org)\), la commande génère une erreur et ne modifie rien dans le workspace.

Nous vous recommandons de ne pas forcer par défaut la mise à jour de toutes les dépendances.
Essayez d'abord de mettre à jour des dépendances spécifiques.

Pour en savoir plus sur le fonctionnement de la commande `ng update`, consultez [Update Command](https://github.com/angular/angular-cli/blob/main/docs/specifications/update.md).

Si vous créez une nouvelle version de votre library qui introduit de potentiels changements cassants, vous pouvez fournir un _schematic de mise à jour_ pour permettre à la commande `ng update` de résoudre automatiquement ces changements dans le projet mis à jour.

Par exemple, supposons que vous vouliez mettre à jour la library Angular Material.

```shell
ng update @angular/material
```

Cette commande met à jour à la fois `@angular/material` et sa dépendance `@angular/cdk` dans le `package.json` de votre workspace.
Si l'un de ces packages contient un schematic de mise à jour couvrant la migration de la version existante vers une nouvelle version, la commande exécute ce schematic sur votre workspace.
