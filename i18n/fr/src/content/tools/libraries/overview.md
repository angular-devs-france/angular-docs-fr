<!-- source: 9a0dfceba9ac437342c5fefe1607ec8e0a23333e -->
# Vue d'ensemble des libraries Angular

De nombreuses applications doivent résoudre les mêmes problèmes généraux, comme présenter une interface utilisateur unifiée, présenter des données et permettre la saisie de données.
Les développeurs peuvent créer des solutions génériques pour des domaines particuliers, adaptables pour être réutilisées dans différentes applications.
Une telle solution peut être construite sous forme de _libraries_ Angular, et ces libraries peuvent être publiées et partagées sous forme de _packages npm_.

Une library Angular est un projet Angular qui se distingue d'une application en ce qu'elle ne peut pas s'exécuter seule.
Une library doit être importée et utilisée dans une application.

Les libraries étendent les fonctionnalités de base d'Angular.
Par exemple, pour ajouter les [reactive forms](guide/forms/reactive-forms) à une application, ajoutez le package de la library avec `ng add @angular/forms`, puis importez le `ReactiveFormsModule` depuis la library `@angular/forms` dans le code de votre application.
De même, ajouter la library [service worker](ecosystem/service-workers) à une application Angular est l'une des étapes pour transformer une application en [Progressive Web App](https://developers.google.com/web/progressive-web-apps) \(PWA\).
[Angular Material](https://material.angular.dev) est un exemple de library vaste et polyvalente qui fournit des composants d'interface sophistiqués, réutilisables et adaptables.

Tout développeur d'application peut utiliser ces libraries, et d'autres, publiées sous forme de packages npm par l'équipe Angular ou par des tiers.
Consultez [Utiliser des libraries publiées](tools/libraries/using-libraries).

HELPFUL: Les libraries sont destinées à être utilisées par des applications Angular. Pour ajouter des fonctionnalités Angular à des applications web qui n'utilisent pas Angular, utilisez les [custom elements Angular](guide/elements).

## Créer des libraries {#creating-libraries}

Si vous avez développé des fonctionnalités qui se prêtent à la réutilisation, vous pouvez créer vos propres libraries.
Ces libraries peuvent être utilisées localement dans votre workspace, ou vous pouvez les publier sous forme de [packages npm](reference/configs/npm-packages) pour les partager avec d'autres projets ou d'autres développeurs Angular.
Ces packages peuvent être publiés sur le registre npm, sur un registre npm Enterprise privé ou sur un système privé de gestion de packages compatible avec les packages npm.
Consultez [Créer des libraries](tools/libraries/creating-libraries).

Décider de packager des fonctionnalités sous forme de library est une décision d'architecture. Elle est comparable au choix de faire d'une fonctionnalité un composant ou un service, ou au choix de la portée d'un composant.

Packager des fonctionnalités sous forme de library oblige les artefacts de la library à être découplés de la logique métier de l'application.
Cela peut aider à éviter diverses mauvaises pratiques ou erreurs d'architecture qui peuvent rendre difficiles le découplage et la réutilisation du code à l'avenir.

Placer du code dans une library séparée est plus complexe que de tout mettre simplement dans une seule application.
Cela demande davantage d'investissement en temps et en réflexion pour gérer, maintenir et mettre à jour la library.
Cette complexité peut s'avérer payante quand la library est utilisée dans plusieurs applications.
