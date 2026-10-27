<!-- source: 9ff1f064f8f052ed51ccf7dc47a7250d6d15beca -->
# Tutoriels intégrés à la documentation Angular

- [Fichiers d'un tutoriel](#tutorial-files)
- [Structure des répertoires des tutoriels](#tutorials-directory-structure)
- [Répertoires réservés des tutoriels](#reserved-tutorials-directories)

## Fichiers d'un tutoriel {#tutorial-files}

Un tutoriel se compose de son contenu, de son code source et de sa configuration.

### Contenu : `README.md` {#content-readmemd}

Le contenu du tutoriel doit se trouver dans un fichier `README.md` du répertoire du tutoriel.

En prenant le tutoriel `learn-angular` comme exemple, consultez : [`adev/src/content/tutorials/learn-angular/intro/README.md`](/adev/src/content/tutorials/learn-angular/intro/README.md)

### Configuration : `config.json` {#configuration-configjson}

Chaque tutoriel est défini par un fichier `config.json`, qui peut contenir les options suivantes :

- `title` : définit le titre du tutoriel utilisé dans la navigation des tutoriels
- `nextTutorial` : le chemin du tutoriel suivant (uniquement dans l'étape `intro/`)
- `src` : le chemin relatif vers un répertoire externe qui définit le code source du tutoriel utilisé dans l'éditeur intégré
- `answerSrc` : le chemin relatif vers un répertoire externe qui définit la réponse du tutoriel utilisée dans l'éditeur intégré
- `openFiles` : un tableau des fichiers à ouvrir dans l'éditeur
- `type` : le type indique comment le tutoriel sera présenté et quels composants sont nécessaires pour ce tutoriel
  - `cli` : un tutoriel de type `cli` contient uniquement le contenu et un terminal interactif avec l'Angular CLI
  - `editor` : utilisé pour l'éditeur intégré complet, qui contient l'éditeur de code, l'aperçu, un terminal interactif et la console avec les sorties du serveur de développement
  - `local` : désactive l'éditeur intégré et n'affiche que le contenu
  - `editor-only` : une configuration spéciale utilisée pour le Playground des tutoriels et le Playground de la page d'accueil, qui désactive le contenu et n'affiche que l'éditeur intégré

### Code source {#source-code}

Le code source du tutoriel comprend tous les fichiers du répertoire du tutoriel, sauf `README.md` et `config.json`.

Le code source du tutoriel a priorité sur les fichiers du projet [`common`](#common) : si un fichier ayant le même chemin relatif existe à la fois dans [`common`](#common) et dans le répertoire du tutoriel, le fichier du tutoriel remplace celui de [`common`](#common).

## Structure des répertoires des tutoriels {#tutorials-directory-structure}

Un tutoriel se compose d'une introduction et d'étapes. L'introduction comme chaque étape contiennent leur propre contenu, leur configuration et leur code source.

En prenant le tutoriel `learn-angular` comme exemple :

### Introduction {#introduction}

[`adev/src/content/tutorials/learn-angular/intro`](/adev/src/content/tutorials/learn-angular/intro)

est l'introduction du tutoriel, qui sera servie sur la route `/tutorials/learn-angular`.

### Étapes {#steps}

[`adev/src/content/tutorials/learn-angular/steps`](/adev/src/content/tutorials/learn-angular/steps) est le répertoire qui contient les étapes du tutoriel.

Voici quelques exemples tirés du tutoriel `learn-angular` :

- [`learn-angular/steps/1-components-in-angular`](/adev/src/content/tutorials/learn-angular/steps/1-components-in-angular) : la route sera `/tutorials/learn-angular/1-components-in-angular`
- [`learn-angular/steps/2-updating-the-component-class`](/adev/src/content/tutorials/learn-angular/steps/2-updating-the-component-class) : la route sera `/tutorials/learn-angular/2-updating-the-component-class`

Chaque répertoire d'étape doit commencer par un nombre suivi d'un tiret, puis du nom de chemin de l'étape.

- Le nombre indique l'étape et détermine l'étape précédente et l'étape suivante au sein d'un tutoriel.
- Le tiret est un délimiteur :).
- Le nom du répertoire, nombre et tiret compris, définit l'URL de l'étape.

## Répertoires réservés des tutoriels {#reserved-tutorials-directories}

### `common`

Le projet common est un projet Angular complet, réutilisé par tous les tutoriels. Il contient toutes
les dépendances (`package.json`, `package-lock.json`), la configuration du projet (`tsconfig.json`, `angular.json`) et les fichiers principaux pour le bootstrap de l'application (`index.html`, `main.ts`, `app.config.ts`).

Un projet commun est utilisé pour plusieurs raisons :

- Éviter la duplication de fichiers dans les tutoriels.
- Optimiser les performances dans l'application en ne demandant qu'une seule fois les fichiers et les dépendances du projet commun, et en profitant du
  cache du navigateur lors des requêtes suivantes.
- N'exiger qu'un seul `npm install` pour tous les tutoriels, ce qui réduit le délai avant de pouvoir interagir avec le tutoriel
  lors de la navigation entre différents tutoriels et étapes.
- Fournir un environnement cohérent à tous les tutoriels.
- Permettre à chaque tutoriel de se concentrer sur le code source propre à ce qu'il enseigne, et non sur la configuration du projet.

Consultez [`adev/shared-docs/pipeline/tutorials/common`](/adev/shared-docs/pipeline/tutorials/common)

Un tutoriel peut aussi fournir son propre répertoire `common`, dont les fichiers sont appliqués par-dessus le projet commun partagé pour l'introduction et chaque étape de ce tutoriel. Consultez [`adev/src/content/tutorials/signals/common`](/adev/src/content/tutorials/signals/common) pour un exemple.

### `playground`

Le répertoire playground contient le code source du Playground des tutoriels, accessible sur `/playground`. Il ne doit contenir aucun contenu.

Consultez [`adev/src/content/tutorials/playground`](/adev/src/content/tutorials/playground)

### `homepage`

Le répertoire homepage contient le code source du Playground de la page d'accueil. Il ne doit contenir aucun contenu.

Consultez [`adev/src/content/tutorials/homepage`](/adev/src/content/tutorials/homepage)

## Mettre à jour les dépendances {#update-dependencies}

Pour mettre à jour les dépendances de tous les tutoriels, vous pouvez exécuter le script suivant

```bash
rm ./adev/src/content/tutorials/homepage/package-lock.json  ./adev/src/content/tutorials/first-app/common/package-lock.json ./adev/src/content/tutorials/learn-angular/common/package-lock.json ./adev/src/content/tutorials/playground/common/package-lock.json ./adev/src/content/tutorials/deferrable-views/common/package-lock.json ./adev/src/content/tutorials/signals/common/package-lock.json ./adev/src/content/tutorials/signal-forms/common/package-lock.json ./adev/shared-docs/pipeline/tutorials/common/package-lock.json

npm i --package-lock-only --prefix ./adev/src/content/tutorials/homepage
npm i --package-lock-only --prefix ./adev/src/content/tutorials/first-app/common
npm i --package-lock-only --prefix ./adev/src/content/tutorials/learn-angular/common
npm i --package-lock-only --prefix ./adev/src/content/tutorials/playground/common
npm i --package-lock-only --prefix ./adev/src/content/tutorials/deferrable-views/common
npm i --package-lock-only --prefix ./adev/src/content/tutorials/signals/common
npm i --package-lock-only --prefix ./adev/src/content/tutorials/signal-forms/common
npm i --package-lock-only --prefix ./adev/shared-docs/pipeline/tutorials/common
```
