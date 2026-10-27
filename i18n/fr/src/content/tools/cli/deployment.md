<!-- source: d69942f76e3394dda3a895dc1510c7536348422a -->
# Déploiement

Lorsque vous êtes prêt à déployer votre application Angular sur un serveur distant, plusieurs options s'offrent à vous.

## Déploiement automatique avec la CLI {#automatic-deployment-with-the-cli}

La commande `ng deploy` de l'Angular CLI exécute le [builder CLI](tools/cli/cli-builder) `deploy` associé à votre projet.
De nombreux builders tiers implémentent des capacités de déploiement vers différentes plateformes.
Vous pouvez ajouter n'importe lequel d'entre eux à votre projet avec `ng add`.

Lorsque vous ajoutez un package doté d'une capacité de déploiement, il met automatiquement à jour la configuration de votre workspace (fichier `angular.json`) avec une section `deploy` pour le projet sélectionné.
Vous pouvez ensuite utiliser la commande `ng deploy` pour déployer ce projet.

Par exemple, la commande suivante déploie automatiquement un projet sur [Firebase](https://firebase.google.com/).

```shell

ng add @angular/fire
ng deploy

```

La commande est interactive.
Dans ce cas, vous devez disposer d'un compte Firebase (ou en créer un) et vous authentifier avec celui-ci.
La commande vous invite à sélectionner un projet Firebase pour le déploiement avant de compiler votre application et d'envoyer les fichiers de production vers Firebase.

Le tableau ci-dessous liste des outils qui implémentent le déploiement vers différentes plateformes.
La commande `deploy` de chaque package peut nécessiter des options de ligne de commande différentes.
Pour en savoir plus, suivez les liens associés aux noms des packages ci-dessous :

| Déploiement vers                                                  | Commande d'installation                                                                     |
| :---------------------------------------------------------------- | :------------------------------------------------------------------------------------------ |
| [Firebase hosting](https://firebase.google.com/docs/hosting)      | [`ng add @angular/fire`](https://www.npmjs.com/package/@angular/fire)                       |
| [Vercel](https://vercel.com/solutions/angular)                    | [`vercel init angular`](https://github.com/vercel/vercel/tree/main/examples/angular)        |
| [Netlify](https://www.netlify.com)                                | [`ng add @netlify-builder/deploy`](https://www.npmjs.com/package/@netlify-builder/deploy)   |
| [GitHub Pages](https://pages.github.com)                          | [`ng add angular-cli-ghpages`](https://www.npmjs.com/package/angular-cli-ghpages)           |
| [Amazon Cloud S3](https://aws.amazon.com/s3/?nc2=h_ql_prod_st_s3) | [`ng add @jefiozie/ngx-aws-deploy`](https://www.npmjs.com/package/@jefiozie/ngx-aws-deploy) |

Si vous déployez sur un serveur autogéré ou s'il n'existe pas de builder pour votre plateforme cloud préférée, vous pouvez soit [créer un builder](tools/cli/cli-builder) qui vous permet d'utiliser la commande `ng deploy`, soit parcourir ce guide pour apprendre à déployer manuellement votre application.

## Déploiement manuel sur un serveur distant {#manual-deployment-to-a-remote-server}

Pour déployer manuellement votre application, créez un build de production et déployez les fichiers générés dans votre environnement d'hébergement.
Par défaut, `ng build` utilise la configuration `production`.
Si vous avez personnalisé vos configurations de build, vérifiez que les [optimisations de production](tools/cli/deployment#production-optimizations) sont bien appliquées avant de déployer.

`ng build` génère par défaut les fichiers compilés dans `dist/my-app/`, mais ce chemin peut être configuré avec l'option `outputPath` du builder `@angular/build:application`.
Ce que vous déployez dépend de la manière dont votre application est rendue :

- **Hébergement statique (rendu côté client ou `outputMode: "static"`) :** copiez le répertoire `dist/my-app/browser/` sur un serveur web ou un réseau de diffusion de contenu (CDN), et configurez-le pour servir ce répertoire.
- **Rendu côté serveur (`outputMode: "server"`) :** copiez les répertoires `dist/my-app/browser/` et `dist/my-app/server/` sur le serveur, puis exécutez `node dist/my-app/server/server.mjs`.

Pour plus de détails, consultez [Rendu serveur et hybride](guide/ssr).

Même s'il s'agit d'une solution de déploiement minimale, le serveur doit respecter quelques exigences pour servir correctement votre application Angular.

## Configuration du serveur {#server-configuration}

Cette section présente les modifications que vous pourriez devoir configurer sur le serveur pour exécuter votre application Angular.

### Les applications avec routing doivent se replier sur `index.html` {#routed-apps-must-fall-back-to-indexhtml}

Les applications Angular rendues côté client sont des candidates idéales pour être servies par un serveur HTML statique, car tout le contenu est statique et généré au moment du build.

Si l'application utilise le router Angular, vous devez configurer le serveur pour qu'il renvoie la page hôte de l'application (`index.html`) lorsqu'on lui demande un fichier qu'il ne possède pas.

Une application avec routing doit prendre en charge les « liens profonds » (deep links).
Un _lien profond_ est une URL qui indique un chemin vers un composant à l'intérieur de l'application.
Par exemple, `http://my-app.test/users/42` est un _lien profond_ vers la page de détail de l'utilisateur qui affiche l'utilisateur dont l'`id` est 42.

Il n'y a aucun problème lorsque l'utilisateur charge d'abord la page d'index puis navigue vers cette URL depuis un client en cours d'exécution.
Le router Angular effectue la navigation _côté client_ et ne demande pas de nouvelle page HTML.

En revanche, cliquer sur un lien profond dans un e-mail, le saisir dans la barre d'adresse du navigateur, ou même actualiser le navigateur alors que vous êtes déjà sur la page du lien profond, sont autant d'actions gérées par le navigateur lui-même, _en dehors_ de l'application en cours d'exécution.
Le navigateur envoie une requête directe au serveur pour `/users/42`, en contournant le router d'Angular.

Un serveur statique renvoie habituellement `index.html` lorsqu'il reçoit une requête pour `http://my-app.test/`.
Mais la plupart des serveurs rejettent par défaut `http://my-app.test/users/42` et renvoient une erreur `404 - Not Found`, _sauf_ s'ils sont configurés pour renvoyer `index.html` à la place.
Configurez la route de repli ou la page 404 de votre serveur vers `index.html`, afin qu'Angular soit servi pour les liens profonds et puisse afficher la bonne route.
Certains serveurs appellent ce comportement de repli le mode « Single-Page Application » (SPA).

Une fois que le navigateur a chargé l'application, le router Angular lit l'URL pour déterminer sur quelle page il se trouve et affiche correctement `/users/42`.

Pour les « vraies » pages 404 comme `http://my-app.test/does-not-exist`, le serveur ne nécessite aucune configuration supplémentaire.
Les [pages 404 implémentées dans le router Angular](guide/routing/common-router-tasks#displaying-a-404-page) s'afficheront correctement.

### Demander des données à un autre serveur (CORS) {#requesting-data-from-a-different-server-cors}

Les développeurs web peuvent rencontrer une erreur de [_partage de ressources entre origines multiples_](https://developer.mozilla.org/docs/Web/HTTP/CORS 'Cross-origin resource sharing') lorsqu'ils effectuent une requête réseau vers un serveur autre que le serveur hôte de l'application.
Les navigateurs interdisent ces requêtes, sauf si le serveur les autorise explicitement.

Ni Angular ni l'application cliente ne peuvent rien faire contre ces erreurs.
C'est le _serveur_ qui doit être configuré pour accepter les requêtes de l'application.
Découvrez comment activer CORS pour des serveurs spécifiques sur [enable-cors.org](https://enable-cors.org/server.html 'Enabling CORS server').

## Optimisations de production {#production-optimizations}

`ng build` utilise la configuration `production`, sauf configuration contraire. Cette configuration active les fonctionnalités d'optimisation du build suivantes.

| Fonctionnalités                                                   | Détails                                                                                       |
| :---------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| [Compilation ahead-of-time (AOT)](tools/cli/aot-compiler)         | Précompile les templates des composants Angular.                                              |
| [Mode production](tools/cli/deployment#development-only-features) | Optimise l'application pour les meilleures performances à l'exécution                         |
| Bundling                                                          | Concatène les nombreux fichiers de votre application et de chaque library en un nombre minimal de fichiers déployés. |
| Minification                                                      | Supprime les espaces superflus, les commentaires et les tokens facultatifs.                   |
| Mangling                                                          | Renomme les fonctions, les classes et les variables avec des identifiants plus courts et arbitraires. |
| Élimination du code mort                                          | Supprime les modules non référencés et le code inutilisé.                                     |

Consultez [`ng build`](cli/build) pour en savoir plus sur les options de build de la CLI et leurs effets.

### Fonctionnalités réservées au développement {#development-only-features}

Lorsque vous exécutez une application en local avec `ng serve`, Angular utilise la configuration de développement
à l'exécution, ce qui active :

- Des vérifications de sécurité supplémentaires, comme la détection [`expression-changed-after-checked`](errors/NG0100).
- Des messages d'erreur plus détaillés.
- Des utilitaires de débogage supplémentaires, comme la variable globale `ng` avec des [fonctions de débogage](api#angular_core_globals) et la prise en charge d'[Angular DevTools](tools/devtools).

Ces fonctionnalités sont utiles pendant le développement, mais elles nécessitent du code supplémentaire dans l'application, ce qui
n'est pas souhaitable en production. Pour que ces fonctionnalités n'aient pas d'impact négatif sur la taille du bundle pour les utilisateurs finaux, l'Angular CLI
supprime le code réservé au développement du bundle lors d'un build de production.

Compiler votre application avec `ng build` utilise par défaut la configuration `production`, qui retire ces fonctionnalités de la sortie pour une taille de bundle optimale.

## `--deploy-url` {#--deploy-url}

`--deploy-url` est une option de ligne de commande qui permet de spécifier le chemin de base utilisé pour résoudre les URL relatives des ressources comme les images, les scripts et les feuilles de style, _à la compilation_.

```shell

ng build --deploy-url /my/assets

```

L'effet et l'objectif de `--deploy-url` recoupent ceux de [`<base href>`](guide/routing/router-reference#base-href). Les deux peuvent être utilisés pour les scripts initiaux, les feuilles de style, les scripts chargés en lazy loading et les ressources CSS.

Contrairement à `<base href>`, qui peut être défini à un seul endroit à l'exécution, `--deploy-url` doit être codé en dur dans l'application au moment du build.
Privilégiez `<base href>` lorsque c'est possible.
