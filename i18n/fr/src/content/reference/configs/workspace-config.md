<!-- source: a9fe9437a4dcd165a9b9d0f72cd5c05feef22e69 -->
# Configuration du workspace Angular

Le fichier `angular.json`, situé au niveau racine d'un workspace Angular, fournit des valeurs de configuration par défaut communes au workspace et spécifiques à chaque projet. Elles sont utilisées par les outils de build et de développement fournis par l'Angular CLI.
Les chemins indiqués dans la configuration sont relatifs au répertoire racine du workspace.

## Structure JSON générale {#general-json-structure}

Au niveau racine de `angular.json`, quelques propriétés configurent le workspace et une section `projects` contient les autres options de configuration propres à chaque projet.
Vous pouvez remplacer les valeurs par défaut de l'Angular CLI définies au niveau du workspace par des valeurs par défaut définies au niveau du projet.
Vous pouvez aussi remplacer les valeurs par défaut définies au niveau du projet depuis la ligne de commande.

Les propriétés suivantes, au niveau racine du fichier, configurent le workspace.

| Propriétés | Détails |
| :--------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version` | La version du fichier de configuration. |
| `newProjectRoot` | Chemin où sont créés les nouveaux projets via des outils comme `ng generate application` ou `ng generate library`. Le chemin peut être absolu ou relatif au répertoire du workspace. Vaut `projects` par défaut |
| `cli` | Un ensemble d'options qui personnalisent l'[Angular CLI](tools/cli). Consultez [Options de configuration de l'Angular CLI](#angular-cli-configuration-options) ci-dessous. |
| `schematics` | Un ensemble de [schematics](tools/cli/schematics) qui personnalisent les valeurs par défaut des options de la sous-commande `ng generate` pour ce workspace. Consultez [Schematics](#schematics) ci-dessous. |
| `projects` | Contient une sous-section pour chaque application ou library du workspace, avec les options de configuration propres au projet. |

L'application initiale que vous créez avec `ng new app-name` est listée sous « projects » :

Lorsque vous créez un projet de library avec `ng generate library`, ce projet de library est aussi ajouté à la section `projects`.

HELPFUL: La section `projects` du fichier de configuration ne correspond pas exactement à la structure de fichiers du workspace.

<!-- markdownlint-disable-next-line MD032 -->

- L'application initiale créée par `ng new` se trouve au niveau racine de la structure de fichiers du workspace.
- Les autres applications et libraries se trouvent par défaut dans le répertoire `projects`.

Pour plus d'informations, consultez [Structure des fichiers du workspace et des projets](reference/configs/file-structure).

## Options de configuration de l'Angular CLI {#angular-cli-configuration-options}

Les propriétés suivantes forment un ensemble d'options qui personnalisent l'Angular CLI.

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :--------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------ | :------------ |
| `analytics` | Partage des données d'utilisation anonymes avec l'équipe Angular. Une valeur booléenne indique s'il faut partager les données ou non, tandis qu'une chaîne UUID partage les données avec un identifiant pseudonyme. | `boolean` \| `string` | `false` |
| `cache` | Contrôle le [cache disque persistant](cli/cache) utilisé par les [builders de l'Angular CLI](tools/cli/cli-builder). | [Options du cache](#cache-options) | `{}` |
| `schematicCollections` | Liste des collections de schematics à utiliser dans `ng generate`. | `string[]` | `[]` |
| `packageManager` | Le gestionnaire de packages à utiliser de préférence. | `npm` \| `pnpm` \| `yarn` \| `bun` | `npm` |
| `warnings` | Contrôle les avertissements de console propres à l'Angular CLI. | [Options des avertissements](#warnings-options) | `{}` |

### Options du cache {#cache-options}

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------- | :--------------- |
| `enabled` | Indique si le cache disque est activé pour les builds. | `boolean` | `true` |
| `environment` | Indique dans quel environnement le cache disque est activé.<br><br>_ `ci` active la mise en cache uniquement dans les environnements d'intégration continue (CI).<br>_ `local` active la mise en cache uniquement _en dehors_ des environnements de CI.<br>\* `all` active la mise en cache partout. | `local` \| `ci` \| `all` | `local` |
| `path` | Le répertoire utilisé pour stocker les résultats du cache. | `string` | `.angular/cache` |

### Options des avertissements {#warnings-options}

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :---------------- | :------------------------------------------------------------------------------ | :--------- | :------------ |
| `versionMismatch` | Affiche un avertissement lorsque la version globale de l'Angular CLI est plus récente que la version locale. | `boolean` | `true` |

## Options de configuration des projets {#project-configuration-options}

Les propriétés de configuration de premier niveau suivantes sont disponibles pour chaque projet, sous `projects['project-name']`.

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------- | :-------------- |
| `root` | Le répertoire racine des fichiers de ce projet, relatif au répertoire du workspace. Vide pour l'application initiale, qui se trouve au niveau racine du workspace. | `string` | Aucune (obligatoire) |
| `projectType` | « application » ou « library ». Une application peut s'exécuter de façon autonome dans un navigateur, contrairement à une library. | `application` \| `library` | Aucune (obligatoire) |
| `sourceRoot` | Le répertoire racine des fichiers source de ce projet. | `string` | `''` |
| `prefix` | Une chaîne qu'Angular ajoute au début des selectors lors de la génération de nouveaux composants, directives et pipes avec `ng generate`. Peut être personnalisée pour identifier une application ou un domaine fonctionnel. | `string` | `'app'` |
| `i18n` | Options d'internationalisation du projet. Définit la locale source et les locales supplémentaires à compiler. Consultez [Définir les locales dans la configuration de build](guide/i18n/merge#define-locales-in-the-build-configuration). | Voir [Options i18n](#i18n-options) | `{}` |
| `schematics` | Un ensemble de schematics qui personnalisent les valeurs par défaut des options de la sous-commande `ng generate` pour ce projet. Consultez la section [Schematics de génération](#schematics). | Voir [Schematics](#schematics) | `{}` |
| `architect` | Valeurs de configuration par défaut des cibles de builders Architect pour ce projet. | Voir [Configurer les cibles de builders](#configuring-builder-targets) | `{}` |

## Options i18n {#i18n-options}

Utilisez l'option de projet `i18n` pour définir la locale source de l'application et les éventuelles locales supplémentaires à compiler.

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------ | :------------ |
| `sourceLocale` | La locale utilisée dans le code source de l'application. Peut être une chaîne d'identifiant de locale ou un [objet de configuration](#sourcelocale-object). | `string` \| [objet sourceLocale](#sourcelocale-object) | `"en-US"` |
| `locales` | Une table associant des identifiants de locale à des fichiers de traduction ou à des [objets de configuration de locale](#locale-object). | `object` | `{}` |

### Objet `sourceLocale` {#sourcelocale-object}

Passez un objet plutôt qu'une chaîne pour personnaliser le répertoire de sortie ou la base HREF de la locale source :

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :--------- | :-------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :------------ |
| `code` | L'identifiant de la locale source. | `string` | `"en-US"` |
| `baseHref` | Remplace le `<base href>` HTML pour cette locale. Le nom du répertoire de sortie reste le code de la locale. Ne peut pas être utilisé avec `subPath`. | `string` | Code de la locale |
| `subPath` | Définit à la fois le nom du répertoire de sortie et le `<base href>` HTML pour cette locale. Ne peut pas être utilisé avec `baseHref`. | `string` | Code de la locale |

### Objet locale {#locale-object}

Chaque entrée de `locales` peut être une chaîne de chemin, un tableau de chemins ou un objet :

| Propriété | Détails | Type de valeur | Valeur par défaut |
| :------------ | :-------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------- | :------------ |
| `translation` | Chemin ou chemins du ou des fichiers de traduction de cette locale. | `string` \| `string[]` | |
| `baseHref` | Remplace le `<base href>` HTML pour cette locale. Le nom du répertoire de sortie reste l'identifiant de la locale. Ne peut pas être utilisé avec `subPath`. | `string` | Code de la locale |
| `subPath` | Définit à la fois le nom du répertoire de sortie et le `<base href>` HTML pour cette locale. Ne peut pas être utilisé avec `baseHref`. | `string` | Code de la locale |

HELPFUL: Utilisez `subPath` plutôt que `baseHref` lorsque vous devez aussi renommer le répertoire de sortie, par exemple pour produire la sortie dans `de-DE/` au lieu de `de/`.

## Schematics {#schematics}

Les [schematics Angular](tools/cli/schematics) sont des instructions qui modifient un projet en ajoutant de nouveaux fichiers ou en modifiant des fichiers existants.
Vous pouvez les configurer en associant le nom du schematic à un ensemble d'options par défaut.

Le « nom » d'un schematic suit le format `<schematic-package>:<schematic-name>`.
Les schematics des sous-commandes `ng generate` par défaut de l'Angular CLI sont regroupés dans le package [`@schematics/angular`](https://github.com/angular/angular-cli/blob/main/packages/schematics/angular/application/schema.json).
Par exemple, le schematic qui génère un composant avec `ng generate component` est `@schematics/angular:component`.

Les champs indiqués dans le schéma du schematic correspondent aux valeurs d'arguments de ligne de commande autorisées et aux valeurs par défaut des options de la sous-commande de l'Angular CLI.
Vous pouvez modifier le fichier de schéma de votre workspace pour définir une autre valeur par défaut pour une option de sous-commande. Par exemple, pour désactiver `standalone` par défaut dans `ng generate component` :

```json
{
  "projects": {
    "my-app": {
      "schematics": {
        "@schematics/angular:component": {
          "standalone": false
        }
      }
    }
  }
}
```

## Configurer les builders de la CLI {#configuring-cli-builders}

Architect est l'outil qu'utilise l'Angular CLI pour effectuer des tâches complexes, comme la compilation et l'exécution des tests.
Architect est une enveloppe qui exécute un builder donné pour effectuer une tâche précise, selon la configuration d'une cible.
Vous pouvez définir et configurer de nouveaux builders et de nouvelles cibles pour étendre l'Angular CLI.
Consultez [Builders de l'Angular CLI](tools/cli/cli-builder).

### Builders et cibles Architect par défaut {#default-architect-builders-and-targets}

Angular définit des builders par défaut à utiliser avec des commandes précises, ou avec la commande générale `ng run`.
Les schémas JSON qui définissent les options et les valeurs par défaut de chacun de ces builders sont regroupés dans le package [`@angular/build`](https://github.com/angular/angular-cli/blob/main/packages/angular/build/builders.json).
Ces schémas configurent les options des builders suivants.

### Configurer les cibles de builders {#configuring-builder-targets}

La section `architect` de `angular.json` contient un ensemble de cibles Architect.
Beaucoup de ces cibles correspondent aux commandes de l'Angular CLI qui les exécutent.
D'autres cibles peuvent être exécutées avec la commande `ng run`, et vous pouvez définir vos propres cibles.

Chaque objet cible indique le `builder` de cette cible, c'est-à-dire le package npm de l'outil qu'exécute Architect.
Chaque cible possède aussi une section `options` qui configure les options par défaut de la cible, et une section `configurations` qui nomme et définit des configurations alternatives pour la cible.
Consultez l'exemple de [Cible de build](#build-target) ci-dessous.

| Propriété | Détails |
| :------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `build` | Configure les valeurs par défaut des options de la commande `ng build`. Consultez la section [Cible de build](#build-target) pour plus d'informations. |
| `serve` | Remplace les valeurs par défaut du build et fournit des valeurs par défaut supplémentaires pour la commande `ng serve`. En plus des options disponibles pour la commande `ng build`, elle ajoute des options liées au service de l'application. |
| `e2e` | Remplace les valeurs par défaut du build pour compiler des applications de test de bout en bout avec la commande `ng e2e`. |
| `test` | Remplace les valeurs par défaut du build pour les builds de test et fournit des valeurs par défaut supplémentaires d'exécution des tests pour la commande `ng test`. |
| `lint` | Configure les valeurs par défaut des options de la commande `ng lint`, qui effectue une analyse statique du code des fichiers source du projet. |
| `extract-i18n` | Configure les valeurs par défaut des options de la commande `ng extract-i18n`, qui extrait les chaînes de messages localisées du code source et produit des fichiers de traduction pour l'internationalisation. |

HELPFUL: Toutes les options du fichier de configuration doivent utiliser le `camelCase`, et non le `dash-case` utilisé en ligne de commande.

## Cible de build {#build-target}

Chaque cible sous `architect` possède les propriétés suivantes :

| Propriété | Détails |
| :--------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `builder` | Le builder CLI utilisé pour créer cette cible, sous la forme `<package-name>:<builder-name>`. |
| `options` | Les options par défaut de la cible de build. |
| `configurations` | Configurations alternatives pour exécuter la cible. Chaque configuration définit les options par défaut pour l'environnement visé, en remplaçant la valeur associée sous `options`. Consultez [Configurations de build alternatives](#alternate-build-configurations) ci-dessous. |

Par exemple, pour configurer un build avec les optimisations désactivées :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "optimization": false
          }
        }
      }
    }
  }
}
```

### Configurations de build alternatives {#alternate-build-configurations}

L'Angular CLI fournit deux configurations de build : `production` et `development`.
Par défaut, la commande `ng build` utilise la configuration `production`, qui applique plusieurs optimisations de build, notamment :

- Le regroupement des fichiers en bundles
- La réduction des espaces superflus
- La suppression des commentaires et du code mort
- La minification du code avec des noms courts et obscurcis

Vous pouvez définir et nommer d'autres configurations alternatives (comme `staging`, par exemple) adaptées à votre processus de développement.
Vous pouvez sélectionner une configuration alternative en passant son nom à l'option de ligne de commande `--configuration`.

Par exemple, pour configurer un build dans lequel l'optimisation n'est activée que pour les builds de production (`ng build --configuration production`) :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "optimization": false
          },
          "configurations": {
            "production": {
              "optimization": true
            }
          }
        }
      }
    }
  }
}
```

Vous pouvez aussi passer plusieurs noms de configuration sous forme de liste séparée par des virgules.
Par exemple, pour appliquer à la fois les configurations de build `staging` et `french`, utilisez la commande `ng build --configuration staging,french`.
Dans ce cas, la commande analyse les configurations nommées de gauche à droite.
Si plusieurs configurations modifient le même paramètre, la dernière valeur définie l'emporte.
Dans cet exemple, si les configurations `staging` et `french` définissent toutes deux le chemin de sortie, c'est la valeur de `french` qui est utilisée.

### Options supplémentaires de build et de test {#extra-build-and-test-options}

Les options configurables d'un build par défaut ou ciblé correspondent généralement aux options disponibles pour les commandes [`ng build`](cli/build) et [`ng test`](cli/test).
Pour le détail de ces options et de leurs valeurs possibles, consultez la [référence de l'Angular CLI](cli).

| Propriétés d'options | Détails |
| :------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `assets` | Un objet contenant les chemins des ressources statiques à servir avec l'application. Les chemins par défaut pointent vers le répertoire `public` du projet. Pour en savoir plus, consultez la section [Configuration des assets](#assets-configuration). |
| `styles` | Un tableau de fichiers CSS à ajouter au contexte global du projet. L'Angular CLI prend en charge les imports CSS et tous les principaux préprocesseurs CSS. Pour en savoir plus, consultez la section [Configuration des styles et des scripts](#styles-and-scripts-configuration). |
| `stylePreprocessorOptions` | Un objet contenant des paires option-valeur à passer aux préprocesseurs de styles. Pour en savoir plus, consultez la section [Configuration des styles et des scripts](#styles-and-scripts-configuration). |
| `inlineStyleLanguage` | Le langage de feuille de style à utiliser pour les styles inline des composants de l'application. Accepte `css`, `less`, `sass` ou `scss`, et vaut `css` par défaut. |
| `scripts` | Un objet contenant les fichiers JavaScript à ajouter à l'application. Les scripts sont chargés exactement comme si vous les aviez ajoutés dans une balise `<script>` à l'intérieur de `index.html`. Pour en savoir plus, consultez la section [Configuration des styles et des scripts](#styles-and-scripts-configuration). |
| `budgets` | Type de budget de taille et seuils par défaut pour tout ou partie de votre application. Vous pouvez configurer le builder pour qu'il signale un avertissement ou une erreur lorsque la sortie atteint ou dépasse une taille seuil. Consultez [Configurer les budgets de taille](tools/cli/build#configuring-size-budgets). |
| `fileReplacements` | Un objet contenant des fichiers et leurs remplacements à la compilation. Pour en savoir plus, consultez [Configurer des remplacements de fichiers propres à une cible](tools/cli/environments#configure-environment-specific-defaults). |
| `index` | Un document HTML de base qui charge l'application. Pour en savoir plus, consultez [Configuration de l'index](#index-configuration). |
| `security` | Un objet contenant la clé `autoCsp`, qui peut valoir `true` ou `false` |

### Options supplémentaires de serve {#extra-serve-options}

Le serveur de développement dispose de son propre ensemble d'options, qui correspondent généralement aux options disponibles pour la commande [`ng serve`](cli/serve).

| Propriétés d'options | Détails |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `allowedHosts` | Un tableau des hôtes auxquels le serveur de développement répond. Cette option définit l'option Vite du même nom. Pour plus de détails, [consultez la documentation de Vite](https://vite.dev/config/server-options.html#server-allowedhosts) |

## Valeurs de configuration complexes {#complex-configuration-values}

Les options `assets`, `index`, `outputPath`, `styles` et `scripts` acceptent soit de simples chaînes de chemin, soit des objets dotés de champs spécifiques.
Les options `sourceMap` et `optimization` peuvent recevoir une simple valeur booléenne. Elles peuvent aussi recevoir une valeur complexe via le fichier de configuration.

Les sections suivantes détaillent l'utilisation de ces valeurs complexes dans chaque cas.

### Configuration des assets {#assets-configuration}

Chaque configuration de cible `build` peut inclure un tableau `assets` qui liste les fichiers ou dossiers à copier tels quels lors du build de votre projet.
Par défaut, le contenu du répertoire `public/` est copié.

Pour exclure une ressource, vous pouvez la retirer de la configuration des assets.

Vous pouvez configurer plus finement les ressources à copier en les spécifiant sous forme d'objets plutôt que de simples chemins relatifs à la racine du workspace.
Un objet de spécification de ressource peut avoir les champs suivants.

| Champs | Détails |
| :--------------- | :---------------------------------------------------------------------------------------------------------------------------------------- |
| `glob` | Un [node-glob](https://github.com/isaacs/node-glob/blob/main/README.md) utilisant `input` comme répertoire de base. |
| `input` | Un chemin relatif à la racine du workspace. |
| `output` | Un chemin relatif à `outDir`. Pour des raisons de sécurité, l'Angular CLI n'écrit jamais de fichiers en dehors du chemin de sortie du projet. |
| `ignore` | Une liste de globs à exclure. |
| `followSymlinks` | Permet aux motifs glob de suivre les répertoires de liens symboliques, afin de parcourir les sous-répertoires du lien symbolique. Vaut `false` par défaut. |

Par exemple, les objets de ressources suivants copient le contenu de `src/assets/` dans `/assets/` et `src/favicon.ico` à la racine de la sortie.

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "assets": [
              {
                "glob": "**/*",
                "input": "src/assets/",
                "output": "/assets/"
              },
              {
                "glob": "favicon.ico",
                "input": "src/",
                "output": "/"
              }
            ]
          }
        }
      }
    }
  }
}
```

L'exemple suivant utilise le champ `ignore` pour empêcher certains fichiers du répertoire des assets d'être copiés dans le build :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "assets": [
              {
                "glob": "**/*",
                "input": "src/assets/",
                "ignore": ["**/*.svg"],
                "output": "/assets/"
              }
            ]
          }
        }
      }
    }
  }
}
```

### Configuration des styles et des scripts {#styles-and-scripts-configuration}

Une entrée de tableau des options `styles` et `scripts` peut être une simple chaîne de chemin ou un objet qui pointe vers un fichier de point d'entrée supplémentaire.
Le builder associé charge ce fichier et ses dépendances dans un bundle distinct pendant le build.
Avec un objet de configuration, vous pouvez nommer le bundle du point d'entrée à l'aide d'un champ `bundleName`.

Le bundle est injecté par défaut, mais vous pouvez définir `inject` à `false` pour exclure le bundle de l'injection.
Par exemple, les objets suivants créent et nomment un bundle qui contient des styles et des scripts, et l'excluent de l'injection :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "styles": [
              {
                "input": "src/external-module/styles.scss",
                "inject": false,
                "bundleName": "external-module"
              }
            ],
            "scripts": [
              {
                "input": "src/external-module/main.js",
                "inject": false,
                "bundleName": "external-module"
              }
            ]
          }
        }
      }
    }
  }
}
```

#### Options des préprocesseurs de styles {#style-preprocessor-options}

Avec Sass, vous pouvez utiliser la fonctionnalité `includePaths` pour les styles des composants comme pour les styles globaux. Elle vous permet d'ajouter des chemins de base supplémentaires dans lesquels les imports sont recherchés.

Pour ajouter des chemins, utilisez l'option `stylePreprocessorOptions` :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "stylePreprocessorOptions": {
              "includePaths": ["src/style-paths"]
            }
          }
        }
      }
    }
  }
}
```

Les fichiers de ce répertoire, comme `src/style-paths/_variables.scss`, peuvent être importés depuis n'importe quel endroit de votre projet sans chemin relatif :

```scss {header: "src/app/app.scss"}
// A relative path works
@import '../style-paths/variables';

// But now this works as well
@import 'variables';
```

HELPFUL: Vous devez aussi ajouter les styles ou scripts au builder `test` si vous en avez besoin pour les tests unitaires.
Consultez aussi [Utiliser des libraries globales à l'exécution dans votre application](tools/libraries/using-libraries#using-runtime-global-libraries-inside-your-app).

### Configuration de l'optimisation {#optimization-configuration}

L'option `optimization` peut être soit un booléen, soit un objet pour une configuration plus fine.
Cette option active diverses optimisations de la sortie du build, notamment :

- La minification des scripts et des styles
- Le tree-shaking
- L'élimination du code mort
- L'[inlining du CSS critique](/tools/cli/build#critical-css-inlining)
- L'inlining des polices

Plusieurs options permettent d'affiner l'optimisation d'une application.

| Options | Détails | Type de valeur | Valeur par défaut |
| :-------- | :------------------------------------------------------------- | :----------------------------------------------------------------------- | :------------ |
| `scripts` | Active l'optimisation des scripts produits. | `boolean` | `true` |
| `styles` | Active l'optimisation des styles produits. | `boolean` \| [Options d'optimisation des styles](#styles-optimization-options) | `true` |
| `fonts` | Active l'optimisation des polices. Nécessite un accès à internet. | `boolean` \| [Options d'optimisation des polices](#fonts-optimization-options) | `true` |

#### Options d'optimisation des styles {#styles-optimization-options}

| Options | Détails | Type de valeur | Valeur par défaut |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------- | :--------- | :------------ |
| `minify` | Minifie les définitions CSS en supprimant les espaces et commentaires superflus, en fusionnant les identifiants et en réduisant les valeurs. | `boolean` | `true` |
| `inlineCritical` | Extrait et intègre inline les définitions CSS critiques pour améliorer le [First Contentful Paint](https://web.dev/first-contentful-paint). | `boolean` | `true` |
| `removeSpecialComments` | Supprime les commentaires du CSS global qui contiennent `@license` ou `@preserve`, ou qui commencent par `//!` ou `/*!`. | `boolean` | `true` |

#### Options d'optimisation des polices {#fonts-optimization-options}

| Options | Détails | Type de valeur | Valeur par défaut |
| :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------- | :------------ |
| `inline` | Réduit les [requêtes bloquant le rendu](https://web.dev/render-blocking-resources) en intégrant inline les définitions CSS externes de Google Fonts et d'Adobe Fonts dans le fichier HTML index de l'application. Nécessite un accès à internet. | `boolean` | `true` |

Vous pouvez fournir une valeur comme la suivante pour appliquer l'optimisation à l'un ou à l'autre :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "optimization": {
              "scripts": true,
              "styles": {
                "minify": true,
                "inlineCritical": true
              },
              "fonts": true
            }
          }
        }
      }
    }
  }
}
```

### Configuration des source maps {#source-map-configuration}

L'option de builder `sourceMap` peut être soit un booléen, soit un objet pour une configuration plus fine des source maps d'une application.

| Options | Détails | Type de valeur | Valeur par défaut |
| :--------------- | :----------------------------------------------------------- | :--------- | :------------ |
| `scripts` | Produit les source maps de tous les scripts. | `boolean` | `true` |
| `styles` | Produit les source maps de tous les styles. | `boolean` | `true` |
| `vendor` | Résout les source maps des packages tiers. | `boolean` | `false` |
| `hidden` | Omet le lien vers les source maps dans le JavaScript produit. | `boolean` | `false` |
| `sourcesContent` | Inclut le contenu source d'origine des fichiers dans les source maps. | `boolean` | `true` |

L'exemple ci-dessous montre comment basculer une ou plusieurs valeurs pour configurer les source maps produites :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "sourceMap": {
              "scripts": true,
              "styles": false,
              "hidden": true,
              "vendor": true
            }
          }
        }
      }
    }
  }
}
```

HELPFUL: Avec des source maps masquées, les source maps ne sont pas référencées dans le bundle.
Elles sont utiles si vous voulez uniquement que les source maps servent à faire correspondre les stack traces dans les outils de signalement d'erreurs, sans qu'elles apparaissent dans les outils de développement du navigateur.
Notez que même si `hidden` empêche de lier la source map dans le bundle produit, votre processus de déploiement doit veiller à ne pas servir les source maps générées en production, faute de quoi les informations fuitent quand même.

#### Source maps sans contenu source {#source-maps-without-sources-content}

Vous pouvez générer des source maps sans le champ `sourcesContent`, qui contient le code source d'origine.
Cela vous permet de déployer des source maps en production pour un meilleur signalement des erreurs avec les noms source d'origine, tout en protégeant votre code source de toute exposition.

Pour exclure le contenu source des source maps, définissez l'option `sourcesContent` à `false` :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "sourceMap": {
              "scripts": true,
              "styles": true,
              "sourcesContent": false
            }
          }
        }
      }
    }
  }
}
```

### Configuration de l'index {#index-configuration}

Configure la génération de l'index HTML de l'application.

L'option `index` peut être soit une chaîne, soit un objet pour une configuration plus fine.

Lorsque la valeur est fournie sous forme de chaîne, le nom de fichier du chemin indiqué est utilisé pour le fichier généré, qui est créé à la racine du chemin de sortie configuré pour l'application.

#### Options de l'index {#index-options}

| Options | Détails | Type de valeur | Valeur par défaut |
| :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :-------------- |
| `input` | Le chemin d'un fichier à utiliser pour l'index HTML généré de l'application. | `string` | Aucune (obligatoire) |
| `output` | Le chemin de sortie du fichier HTML index généré de l'application. Le chemin complet fourni est utilisé et considéré comme relatif au chemin de sortie configuré pour l'application. | `string` | `index.html` |

### Configuration du chemin de sortie {#output-path-configuration}

L'option `outputPath` peut être soit une chaîne, utilisée comme valeur de `base`, soit un objet pour une configuration plus fine.

Plusieurs options permettent d'affiner la structure de sortie d'une application.

| Options | Détails | Type de valeur | Valeur par défaut |
| :-------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------- | :------------ |
| `base` | Indique le chemin de sortie relatif à la racine du workspace. | `string` | |
| `browser` | Le nom du répertoire de sortie du build navigateur, situé dans le chemin de sortie de base. Il peut être servi aux utilisateurs en toute sécurité. | `string` | `browser` |
| `server` | Le nom du répertoire de sortie du build serveur, situé dans le chemin de sortie de base. | `string` | `server` |
| `media` | Le nom du répertoire de sortie des fichiers média, situé dans le répertoire de sortie navigateur. Ces fichiers média sont couramment appelés ressources dans les fichiers CSS. | `string` | `media` |
