<!-- source: 6834f37e6275cec144bef293d505d12843cb0969 -->
# Présentation du service worker Angular

IMPORTANT: Le service worker Angular est un utilitaire de mise en cache basique, destiné à une prise en charge simple du mode hors ligne, avec un ensemble de fonctionnalités limité. Nous n'accepterons plus de nouvelles fonctionnalités, hormis les correctifs de sécurité. Pour des capacités de mise en cache et de fonctionnement hors ligne plus avancées, nous vous recommandons d'explorer directement les API natives des navigateurs.

Les service workers enrichissent le modèle de déploiement web traditionnel et permettent aux applications d'offrir une expérience utilisateur dont la fiabilité et les performances sont comparables à celles d'un code écrit pour s'exécuter sur votre système d'exploitation et votre matériel.
Ajouter un service worker à une application Angular est l'une des étapes pour transformer une application en [Progressive Web App](https://web.dev/progressive-web-apps/) (aussi appelée PWA).

Dans sa forme la plus simple, un service worker est un script qui s'exécute dans le navigateur web et gère la mise en cache d'une application.

Les service workers fonctionnent comme un proxy réseau.
Ils interceptent toutes les requêtes HTTP sortantes émises par l'application et peuvent choisir comment y répondre.
Par exemple, ils peuvent interroger un cache local et fournir une réponse en cache si elle est disponible.
Ce rôle de proxy ne se limite pas aux requêtes émises via des API programmatiques, comme `fetch` ; il inclut aussi les ressources référencées dans le HTML, et même la requête initiale vers `index.html`.
La mise en cache basée sur un service worker est donc entièrement programmable et ne dépend pas des en-têtes de mise en cache définis par le serveur.

Contrairement aux autres scripts qui composent une application, comme le bundle de l'application Angular, le service worker est conservé après que l'utilisateur a fermé l'onglet.
La prochaine fois que ce navigateur charge l'application, le service worker se charge en premier et peut intercepter chaque requête de ressource nécessaire au chargement de l'application.
Si le service worker est conçu pour cela, il peut _assurer entièrement le chargement de l'application, sans avoir besoin du réseau_.

Même sur un réseau rapide et fiable, les délais d'aller-retour peuvent introduire une latence importante lors du chargement de l'application.
Utiliser un service worker pour réduire la dépendance au réseau peut améliorer considérablement l'expérience utilisateur.

## Les service workers dans Angular {#service-workers-in-angular}

Les applications Angular, en tant que single-page applications, sont idéalement placées pour profiter des avantages des service workers. Angular fournit une implémentation de service worker. Les développeurs Angular peuvent tirer parti de ce service worker et bénéficier de la fiabilité et des performances accrues qu'il apporte, sans avoir à coder avec des API de bas niveau.

Le service worker d'Angular est conçu pour optimiser l'expérience de l'utilisateur final lorsqu'il utilise une application sur une connexion réseau lente ou peu fiable, tout en minimisant les risques de servir du contenu obsolète.

Pour y parvenir, le service worker Angular suit ces principes :

- Mettre en cache une application revient à installer une application native.
  L'application est mise en cache comme un tout, et tous les fichiers sont mis à jour ensemble.

- Une application en cours d'exécution continue de fonctionner avec la même version de tous ses fichiers.
  Elle ne se met pas soudainement à recevoir des fichiers en cache d'une version plus récente, probablement incompatibles.

- Quand les utilisateurs rafraîchissent l'application, ils voient la dernière version entièrement mise en cache.
  Les nouveaux onglets chargent le dernier code mis en cache.

- Les mises à jour se font en arrière-plan, relativement vite après la publication des modifications.
  La version précédente de l'application est servie jusqu'à ce qu'une mise à jour soit installée et prête.

- Le service worker économise la bande passante quand c'est possible.
  Les ressources ne sont téléchargées que si elles ont changé.

Pour assurer ces comportements, le service worker Angular charge un fichier _manifeste_ depuis le serveur.
Ce fichier, appelé `ngsw.json` (à ne pas confondre avec le [manifeste d'application web](https://developer.mozilla.org/docs/Web/Manifest)), décrit les ressources à mettre en cache et inclut des hashes du contenu de chaque fichier.
Lorsqu'une mise à jour de l'application est déployée, le contenu du manifeste change, ce qui informe le service worker qu'une nouvelle version de l'application doit être téléchargée et mise en cache.
Ce manifeste est généré à partir d'un fichier de configuration généré par la CLI, appelé `ngsw-config.json`.

Installer le service worker Angular est aussi simple que [lancer une commande de l'Angular CLI](ecosystem/service-workers/getting-started#adding-a-service-worker-to-your-project).
En plus d'enregistrer le service worker Angular auprès du navigateur, cela rend aussi disponibles à l'injection quelques services qui interagissent avec le service worker et permettent de le contrôler.
Par exemple, une application peut demander à être notifiée quand une nouvelle mise à jour est disponible, ou demander au service worker de vérifier auprès du serveur si des mises à jour sont disponibles.

## Avant de commencer {#before-you-start}

Pour profiter de toutes les fonctionnalités des service workers Angular, utilisez les dernières versions d'Angular et de l'[Angular CLI](tools/cli).

Pour que les service workers soient enregistrés, l'application doit être accessible en HTTPS, et non en HTTP.
Les navigateurs ignorent les service workers sur les pages servies via une connexion non sécurisée.
La raison est que les service workers sont très puissants : il faut donc s'assurer avec un soin particulier que le script du service worker n'a pas été altéré.

Il existe une exception à cette règle : pour simplifier le développement local, les navigateurs n'exigent _pas_ de connexion sécurisée pour accéder à une application sur `localhost`.

### Prise en charge des navigateurs {#browser-support}

Pour bénéficier du service worker Angular, votre application doit s'exécuter dans un navigateur web qui prend en charge les service workers de manière générale.
Actuellement, les service workers sont pris en charge dans les dernières versions de Chrome, Firefox, Edge, Safari, Opera, UC Browser (version Android) et Samsung Internet.
Des navigateurs comme IE et Opera Mini ne prennent pas en charge les service workers.

Si l'utilisateur accède à votre application avec un navigateur qui ne prend pas en charge les service workers, le service worker n'est pas enregistré et les comportements associés, comme la gestion du cache hors ligne et les notifications push, n'ont pas lieu.
Plus précisément :

- Le navigateur ne télécharge pas le script du service worker ni le fichier manifeste `ngsw.json`
- Les tentatives actives d'interagir avec le service worker, comme l'appel à `SwUpdate.checkForUpdate()`, renvoient des promises rejetées
- Les événements émis par l'observable de chaque service associé, comme `SwUpdate.versionUpdates`, ne sont pas déclenchés

Il est fortement recommandé de vous assurer que votre application fonctionne même sans prise en charge des service workers dans le navigateur.
Bien qu'un navigateur non compatible ignore la mise en cache par le service worker, il signale tout de même des erreurs si l'application tente d'interagir avec le service worker.
Par exemple, appeler `SwUpdate.checkForUpdate()` renvoie des promises rejetées.
Pour éviter une telle erreur, vérifiez si le service worker Angular est activé avec `SwUpdate.isEnabled`.

Pour en savoir plus sur les autres navigateurs compatibles avec les service workers, consultez la page [Can I Use](https://caniuse.com/#feat=serviceworkers) et la [documentation MDN](https://developer.mozilla.org/docs/Web/API/Service_Worker_API).

## Ressources associées {#related-resources}

Les autres articles de cette section traitent spécifiquement de l'implémentation des service workers dans Angular.

<docs-pill-row>
  <docs-pill href="ecosystem/service-workers/config" title="Fichier de configuration"/>
  <docs-pill href="ecosystem/service-workers/communications" title="Communiquer avec le service worker"/>
  <docs-pill href="ecosystem/service-workers/push-notifications" title="Notifications push"/>
  <docs-pill href="ecosystem/service-workers/devops" title="DevOps du service worker"/>
  <docs-pill href="ecosystem/service-workers/app-shell" title="Pattern app shell"/>
</docs-pill-row>

Pour plus d'informations sur les service workers en général, consultez [Service Workers: an Introduction](https://developers.google.com/web/fundamentals/primers/service-workers).

Pour plus d'informations sur la prise en charge des navigateurs, consultez la section [browser support](https://developers.google.com/web/fundamentals/primers/service-workers/#browser_support) de [Service Workers: an Introduction](https://developers.google.com/web/fundamentals/primers/service-workers), l'article [Is Serviceworker ready?](https://jakearchibald.github.io/isserviceworkerready) de Jake Archibald, et [Can I Use](https://caniuse.com/serviceworkers).

Pour d'autres recommandations et exemples, consultez :

<docs-pill-row>
  <docs-pill href="https://web.dev/precaching-with-the-angular-service-worker" title="Précache avec le service worker Angular"/>
  <docs-pill href="https://web.dev/creating-pwa-with-angular-cli" title="Créer une PWA avec l'Angular CLI"/>
</docs-pill-row>

## Étape suivante {#next-step}

Pour commencer à utiliser les service workers Angular, consultez [Bien démarrer avec les service workers](ecosystem/service-workers/getting-started).
