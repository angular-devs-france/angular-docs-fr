<!-- source: 7131d3b205717ae5a2b9c5946f0ab49cbe8508d1 -->
# Présentation des DevTools

Angular DevTools est une extension de navigateur qui fournit des fonctionnalités de débogage et de profilage pour les applications Angular.

<docs-video src="https://www.youtube.com/embed/bavWOHZM6zE"/>

Installez Angular DevTools depuis le [Chrome Web Store](https://chrome.google.com/webstore/detail/angular-developer-tools/ienfalfjdbdpebioblfackkekamfmbnh) ou depuis [Firefox Add-ons](https://addons.mozilla.org/firefox/addon/angular-devtools/).

Vous pouvez ouvrir les DevTools de Chrome ou de Firefox sur n'importe quelle page web en appuyant sur <kbd>F12</kbd> ou <kbd><kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>I</kbd></kbd> (Windows ou Linux) et <kbd><kbd>Fn</kbd>+<kbd>F12</kbd></kbd> ou <kbd><kbd>Cmd</kbd>+<kbd>Option</kbd>+<kbd>I</kbd></kbd> (Mac).
Une fois les DevTools du navigateur ouverts et Angular DevTools installé, vous le trouverez dans l'onglet « Angular ».

HELPFUL: La page Nouvel onglet de Chrome n'exécute pas les extensions installées : l'onglet Angular n'apparaîtra donc pas dans les DevTools. Rendez-vous sur n'importe quelle autre page pour le voir.

<img src="assets/images/guide/devtools/devtools.png" alt="Une vue d'ensemble d'Angular DevTools montrant un arbre de composants pour une application.">

## Ouvrir votre application {#open-your-application}

Lorsque vous ouvrez l'extension, vous voyez quatre onglets supplémentaires :

| Onglets                                   | Détails                                                                                                                |
| :---------------------------------------- | :--------------------------------------------------------------------------------------------------------------------- |
| [Components](tools/devtools/component)    | Vous permet d'explorer les composants et les directives de votre application, et de prévisualiser ou modifier leur état. |
| [Profiler](tools/devtools/profiler)       | Vous permet de profiler votre application et de comprendre où se situe le goulot d'étranglement de performances pendant l'exécution de la détection de changements. |
| [Injector Tree](tools/devtools/injectors) | Vous permet de visualiser la hiérarchie des injectors : injector d'environnement et element injector                      |
| [Router Tree](tools/devtools/router)      | Vous permet de visualiser l'arbre de routing de votre application.                                                     |

D'autres onglets comme `Transfer State` sont expérimentaux : ils peuvent être activés dans les paramètres des devtools et ne sont pas encore documentés.

HELPFUL: Si vous utilisez un navigateur basé sur Chromium, l'[intégration au panneau Performance](/best-practices/profiling-with-chrome-devtools) pourrait vous intéresser.

<img src="assets/images/guide/devtools/devtools-tabs.png" alt="Une capture d'écran du haut d'Angular DevTools montrant deux onglets dans le coin supérieur gauche, l'un intitulé 'Components' et l'autre 'Profiler'.">

Dans le coin supérieur droit d'Angular DevTools se trouve le bouton d'information, qui ouvre un popover.
Le popover d'information indique notamment la version d'Angular exécutée sur la page ainsi que la version des devtools.

### Application Angular non détectée {#angular-application-not-detected}

Si vous voyez le message d'erreur « Angular application not detected » en ouvrant Angular DevTools, cela signifie que l'extension ne parvient pas à communiquer avec une application Angular sur la page.
La raison la plus courante est que la page web que vous inspectez ne contient pas d'application Angular.
Vérifiez que vous inspectez la bonne page web et que l'application Angular est en cours d'exécution.

### Une application compilée avec la configuration de production a été détectée {#we-detected-an-application-built-with-production-configuration}

Si vous voyez le message d'erreur « We detected an application built with production configuration. Angular DevTools only supports development builds. », cela signifie qu'une application Angular a été trouvée sur la page, mais qu'elle a été compilée avec les optimisations de production.
Lors de la compilation pour la production, l'Angular CLI supprime diverses fonctionnalités de débogage afin de réduire la quantité de JavaScript sur la page et d'améliorer les performances. Cela inclut les fonctionnalités nécessaires pour communiquer avec les DevTools.

Pour utiliser les DevTools, vous devez compiler votre application avec les optimisations désactivées. `ng serve` le fait par défaut.
Si vous devez déboguer une application déployée, désactivez les optimisations de votre build avec l'[option de configuration `optimization`](reference/configs/workspace-config#optimization-configuration) (`{"optimization": false}`).
