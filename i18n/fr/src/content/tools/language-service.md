<!-- source: d6492cb353dc1f6a6600c2b62a026094f4bfb1b8 -->
# Angular Language Service

L'Angular Language Service permet aux éditeurs de code d'obtenir des complétions, des erreurs, des indications et de la navigation dans les templates Angular.
Il fonctionne avec les templates externes placés dans des fichiers HTML séparés, mais aussi avec les templates inline.

## Configurer les options du compilateur pour l'Angular Language Service {#configuring-compiler-options-for-the-angular-language-service}

Pour activer les dernières fonctionnalités du Language Service, définissez l'option `strictTemplates` dans `tsconfig.json` en passant `strictTemplates` à `true`, comme dans l'exemple suivant :

```json

"angularCompilerOptions": {
  "strictTemplates": true
}

```

Pour plus d'informations, consultez le guide des [options du compilateur Angular](reference/configs/angular-compiler-options).

## Fonctionnalités {#features}

Votre éditeur détecte automatiquement que vous ouvrez un fichier Angular.
Il utilise alors l'Angular Language Service pour lire votre fichier `tsconfig.json`, trouver tous les templates de votre application, puis fournir des services de langage pour chaque template que vous ouvrez.

Les services de langage comprennent :

- Les listes de complétions
- Les messages de diagnostic AOT
- Les infos rapides
- L'accès à la définition

### Autocomplétion {#autocompletion}

L'autocomplétion peut accélérer votre développement en vous proposant des possibilités et des indications contextuelles pendant que vous tapez.
Cet exemple montre l'autocomplétion dans une interpolation.
Pendant la saisie, vous pouvez appuyer sur Tab pour compléter.

<img alt="autocomplétion" src="assets/images/guide/language-service/language-completion.gif">

Des complétions sont aussi proposées au sein des éléments.
Tout élément utilisé comme selector de composant apparaît dans la liste de complétions.

### Vérification des erreurs {#error-checking}

L'Angular Language Service peut vous avertir à l'avance des erreurs dans votre code.
Dans cet exemple, Angular ne sait pas ce qu'est `orders` ni d'où il provient.

<img alt="vérification des erreurs" src="assets/images/guide/language-service/language-error.gif">

### Infos rapides et navigation {#quick-info-and-navigation}

La fonctionnalité d'infos rapides vous permet de voir, au survol, d'où proviennent les composants, les directives et les modules.
Vous pouvez ensuite cliquer sur « Go to definition » ou appuyer sur F12 pour accéder directement à la définition.

<img alt="navigation" src="assets/images/guide/language-service/language-navigation.gif">

## L'Angular Language Service dans votre éditeur {#angular-language-service-in-your-editor}

L'Angular Language Service est actuellement disponible sous forme d'extension pour [Visual Studio Code](https://code.visualstudio.com), [WebStorm](https://www.jetbrains.com/webstorm), [Sublime Text](https://www.sublimetext.com), [Zed](https://zed.dev), [Neovim](https://neovim.io) et [Eclipse IDE](https://www.eclipse.org/eclipseide).

NOTE: L'équipe Angular maintient et distribue l'extension Visual Studio Code, prise en charge dans Visual Studio Code et dans les éditeurs basés sur VS Code comme Cursor et Antigravity. Les intégrations pour les autres éditeurs sont maintenues par leurs éditeurs respectifs ou par la communauté. Signalez les problèmes propres à ces intégrations à leurs mainteneurs.

### Visual Studio Code {#visual-studio-code}

Dans [Visual Studio Code](https://code.visualstudio.com), installez l'extension depuis le [Marketplace des extensions](https://marketplace.visualstudio.com/items?itemName=Angular.ng-template).
Ouvrez le marketplace depuis l'éditeur à l'aide de l'icône Extensions dans le panneau de menu de gauche, ou utilisez VS Quick Open \(⌘+P sur Mac, CTRL+P sur Windows\) et tapez « ? ext ».
Dans le marketplace, recherchez l'extension Angular Language Service et cliquez sur le bouton **Install**.

L'intégration de l'Angular Language Service à Visual Studio Code est maintenue et distribuée par l'équipe Angular.

### Visual Studio {#visual-studio}

Dans [Visual Studio](https://visualstudio.microsoft.com), installez l'extension depuis le [Marketplace des extensions](https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.AngularLanguageService).
Ouvrez le marketplace depuis l'éditeur en sélectionnant Extensions dans le menu du haut, puis Manage Extensions.
Dans le marketplace, recherchez l'extension Angular Language Service et cliquez sur le bouton **Install**.

L'intégration de l'Angular Language Service à Visual Studio est maintenue et distribuée par Microsoft avec l'aide de l'équipe Angular.
Consultez le projet [ici](https://github.com/microsoft/vs-ng-language-service).

### WebStorm {#webstorm}

Dans [WebStorm](https://www.jetbrains.com/webstorm), activez le plugin [Angular and AngularJS](https://plugins.jetbrains.com/plugin/6971-angular-and-angularjs).

Depuis WebStorm 2019.1, `@angular/language-service` n'est plus nécessaire et doit être supprimé de votre `package.json`.

### Sublime Text {#sublime-text}

Dans [Sublime Text](https://www.sublimetext.com), le Language Service ne prend en charge que les templates inline lorsqu'il est installé comme plug-in.
Vous avez besoin d'un plug-in Sublime personnalisé \(ou de modifications du plug-in actuel\) pour obtenir des complétions dans les fichiers HTML.

Pour utiliser le Language Service avec les templates inline, vous devez d'abord ajouter une extension prenant en charge TypeScript, puis installer le plug-in Angular Language Service.
Depuis TypeScript 2.3, TypeScript dispose d'un modèle de plug-in que le Language Service peut utiliser.

1. Installez la dernière version de TypeScript dans un répertoire `node_modules` local :

   ```shell
   npm install --save-dev typescript
   ```

1. Installez le package Angular Language Service au même emplacement :

   ```shell

   npm install --save-dev @angular/language-service

   ```

1. Une fois le package installé, ajoutez le code suivant dans la section `"compilerOptions"` du `tsconfig.json` de votre projet.

   ```json {header:"tsconfig.json"}
   "plugins": [
     {"name": "@angular/language-service"}
   ]
   ```

1. Dans les préférences utilisateur de votre éditeur \(`Cmd+,` ou `Ctrl+,`\), ajoutez le code suivant :

   ```json {header:"Sublime Text user preferences"}

   "typescript-tsdk": "<path to your folder>/node_modules/typescript/lib"

   ```

L'Angular Language Service peut ainsi fournir des diagnostics et des complétions dans les fichiers `.ts`.

### Eclipse IDE {#eclipse-ide}

Installez directement le package « Eclipse IDE for Web and JavaScript developers », qui inclut l'Angular Language Server, ou, depuis d'autres packages Eclipse IDE, utilisez Help > Eclipse Marketplace pour trouver et installer [Eclipse Wild Web Developer](https://marketplace.eclipse.org/content/wild-web-developer-html-css-javascript-typescript-nodejs-angular-json-yaml-kubernetes-xml).

### Neovim {#neovim}

#### Conquer of Completion avec Node.js {#conquer-of-completion-with-nodejs}

L'Angular Language Service utilise tsserver, qui ne suit pas exactement les spécifications LSP. Par conséquent, si vous utilisez neovim ou vim avec JavaScript, TypeScript ou Angular, vous constaterez peut-être que [Conquer of Completion](https://github.com/neoclide/coc.nvim) (COC) offre l'implémentation la plus complète de l'Angular Language Service et de tsserver. En effet, COC porte l'implémentation VSCode de tsserver, qui s'adapte à l'implémentation de tsserver.

1. [Installez coc.nvim](https://github.com/neoclide/coc.nvim)

2. Configurez l'Angular Language Service

   Une fois l'installation terminée, exécutez la commande vim `CocConfig` pour ouvrir le fichier de configuration `coc-settings.json` et ajoutez la propriété angular.

   Veillez à remplacer les chemins par ceux de vos `node_modules` globaux, de sorte qu'ils pointent vers les répertoires qui contiennent respectivement `tsserver` et `ngserver`.

   ```json {header:"CocConfig example file coc-settings.json"}
   {
     "languageserver": {
       "angular": {
         "command": "ngserver",
         "args": [
           "--stdio",
           "--tsProbeLocations",
           "/usr/local/lib/node_modules/typescript/lib/CHANGE/THIS/TO/YOUR/GLOBAL/NODE_MODULES",
           "--ngProbeLocations",
           "/usr/local/lib/node_modules/@angular/language-server/bin/CHANGE/THIS/TO/YOUR/GLOBAL/NODE_MODULES"
         ],
         "filetypes": ["ts", "typescript", "html"],
         "trace.server.verbosity": "verbose"
       }
     }
   }
   ```

HELPFUL: `/usr/local/lib/node_modules/typescript/lib` et `/usr/local/lib/node_modules/@angular/language-server/bin` ci-dessus doivent pointer vers l'emplacement de vos node modules globaux, qui peut être différent.

#### LSP intégré à Neovim {#built-in-neovim-lsp}

L'Angular Language Service peut être utilisé avec Neovim grâce au plugin [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig).

1. [Installez nvim-lspconfig](https://github.com/neovim/nvim-lspconfig?tab=readme-ov-file#install)

2. [Configurez angularls pour nvim-lspconfig](https://github.com/neovim/nvim-lspconfig/blob/master/doc/configs.md#angularls)

### Zed {#zed}

Dans [Zed](https://zed.dev), installez l'extension maintenue par la communauté depuis le [Marketplace des extensions](https://zed.dev/extensions/angular).

## Fonctionnement du Language Service {#how-the-language-service-works}

Lorsque vous utilisez un éditeur doté d'un service de langage, l'éditeur démarre un processus de service de langage distinct et communique avec lui via un [RPC](https://en.wikipedia.org/wiki/Remote_procedure_call), à l'aide du [Language Server Protocol](https://microsoft.github.io/language-server-protocol).
Lorsque vous tapez dans l'éditeur, celui-ci envoie des informations au processus de service de langage pour suivre l'état de votre projet.

Lorsque vous déclenchez une liste de complétions dans un template, l'éditeur commence par analyser le template pour en faire un [arbre syntaxique abstrait (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree) HTML.
Le compilateur Angular interprète cet arbre pour déterminer le contexte : le module dont fait partie le template, la portée actuelle, le selector du composant et la position de votre curseur dans l'AST du template.
Il peut alors déterminer les symboles susceptibles de se trouver à cette position.

C'est un peu plus complexe si vous êtes dans une interpolation.
Si vous avez une interpolation `{{data.---}}` dans une `div` et avez besoin de la liste de complétions après `data.---`, le compilateur ne peut pas utiliser l'AST HTML pour trouver la réponse.
L'AST HTML peut seulement indiquer au compilateur qu'il existe du texte contenant les caractères « `{{data.---}}` ».
C'est alors que le parser de template produit un AST d'expression, qui se trouve dans l'AST du template.
L'Angular Language Service examine ensuite `data.---` dans son contexte, demande au TypeScript Language Service quels sont les membres de `data`, et renvoie la liste des possibilités.

## Plus d'informations {#more-information}

- Pour des informations plus approfondies sur l'implémentation, consultez le [code source de l'Angular Language Service](https://github.com/angular/angular/blob/main/packages/language-service/src)
- Pour en savoir plus sur les choix et les intentions de conception, consultez la [documentation de conception](https://github.com/angular/vscode-ng-language-service/wiki/Design)
