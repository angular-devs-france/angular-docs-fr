<!-- source: 38830ddd548d23f84004e5b3b815caa87c8ea827 -->
# DevOps du service worker

Cette page est une référence pour déployer et maintenir des applications en production qui utilisent le service worker d'Angular.
Elle explique comment le service worker d'Angular s'intègre dans l'environnement de production global, son comportement dans diverses situations, ainsi que les ressources et mécanismes de sécurité disponibles.

## Service worker et mise en cache des ressources de l'application {#service-worker-and-caching-of-application-resources}

Imaginez le service worker d'Angular comme un cache de proxy ou un nœud périphérique de réseau de diffusion de contenu (CDN) installé dans le navigateur web de l'utilisateur final.
Le service worker répond aux requêtes de ressources ou de données émises par l'application Angular à partir d'un cache local, sans avoir à attendre le réseau.
Comme tout cache, il applique des règles d'expiration et de mise à jour du contenu.

### Versions de l'application {#application-versions}

Dans le contexte d'un service worker Angular, une « version » est un ensemble de ressources qui représente un build précis de l'application Angular.
Chaque fois qu'un nouveau build de l'application est déployé, le service worker traite ce build comme une nouvelle version de l'application.
C'est vrai même si un seul fichier a été mis à jour.
À tout moment, le service worker peut avoir plusieurs versions de l'application dans son cache et les servir simultanément.
Pour plus d'informations, consultez la section [Onglets de l'application](#application-tabs).

Pour préserver l'intégrité de l'application, le service worker d'Angular regroupe tous les fichiers dans une même version.
Les fichiers regroupés dans une version incluent généralement les fichiers HTML, JS et CSS.
Ce regroupement est essentiel à l'intégrité, car les fichiers HTML, JS et CSS se font souvent référence les uns aux autres et dépendent d'un contenu précis.
Par exemple, un fichier `index.html` peut contenir une balise `<script>` qui référence `bundle.js` et tenter d'appeler une fonction `startApp()` définie dans ce script.
Chaque fois que cette version d'`index.html` est servie, le `bundle.js` correspondant doit être servi avec elle.
Supposons par exemple que la fonction `startApp()` soit renommée en `runApp()` dans les deux fichiers.
Dans ce scénario, il n'est pas valide de servir l'ancien `index.html`, qui appelle `startApp()`, avec le nouveau bundle, qui définit `runApp()`.

Cette intégrité des fichiers est particulièrement importante avec le lazy loading.
Un bundle JS peut référencer de nombreux chunks chargés en lazy loading, et les noms de fichiers de ces chunks sont propres à chaque build de l'application.
Si une application en cours d'exécution en version `X` tente de charger un chunk en lazy loading alors que le serveur est déjà passé à la version `X + 1`, l'opération de lazy loading échoue.

L'identifiant de version de l'application est déterminé par le contenu de toutes les ressources, et il change si l'une d'elles change.
En pratique, la version est déterminée par le contenu du fichier `ngsw.json`, qui inclut les hashs de tout le contenu connu.
Si l'un des fichiers en cache change, le hash de ce fichier change dans `ngsw.json`. Ce changement amène le service worker d'Angular à traiter l'ensemble de fichiers actif comme une nouvelle version.

HELPFUL: Le processus de build crée le fichier manifeste, `ngsw.json`, à partir des informations de `ngsw-config.json`.

Grâce à ce comportement de gestion des versions du service worker d'Angular, un serveur d'application peut garantir que l'application Angular dispose toujours d'un ensemble de fichiers cohérent.

#### Vérifications de mise à jour {#update-checks}

Chaque fois que l'utilisateur ouvre ou actualise l'application, le service worker d'Angular vérifie s'il existe des mises à jour de l'application en recherchant des mises à jour du manifeste `ngsw.json`.
Si une mise à jour est trouvée, elle est automatiquement téléchargée et mise en cache, puis servie au prochain chargement de l'application.

### Intégrité des ressources {#resource-integrity}

L'un des effets de bord possibles d'une mise en cache de longue durée est de mettre en cache par inadvertance une ressource non valide.
Dans un cache HTTP classique, une actualisation forcée ou l'expiration du cache limite les effets négatifs de la mise en cache d'un fichier non valide.
Un service worker ignore ces contraintes et met en réalité en cache toute l'application sur une longue durée.
Il est important que le service worker obtienne le bon contenu ; il conserve donc les hashs des ressources pour garantir leur intégrité.

#### Contenu hashé {#hashed-content}

Pour garantir l'intégrité des ressources, le service worker d'Angular valide les hashs de toutes les ressources pour lesquelles il dispose d'un hash.
Pour une application créée avec l'[Angular CLI](tools/cli), cela correspond à tout ce qui se trouve dans le répertoire `dist` et qui est couvert par la configuration `src/ngsw-config.json` de l'utilisateur.

Si la validation d'un fichier échoue, le service worker d'Angular tente de récupérer à nouveau le contenu en utilisant un paramètre d'URL de « cache-busting » pour éviter la mise en cache par le navigateur ou par un intermédiaire.
Si ce contenu échoue également à la validation, le service worker considère que toute la version de l'application n'est pas valide et cesse de servir l'application.
Si nécessaire, le service worker passe en mode sans échec, où les requêtes se rabattent sur le réseau. Le service worker n'utilise pas son cache s'il existe un risque élevé de servir un contenu défectueux, obsolète ou non valide.

Des incohérences de hash peuvent survenir pour diverses raisons :

- Des couches de cache entre le serveur d'origine et l'utilisateur final peuvent servir un contenu périmé
- Un déploiement non atomique peut amener le service worker d'Angular à voir un contenu partiellement mis à jour
- Des erreurs pendant le processus de build peuvent produire des ressources mises à jour sans que `ngsw.json` soit mis à jour.
  L'inverse peut aussi se produire, avec un `ngsw.json` mis à jour sans ressources mises à jour.

#### Contenu non hashé {#unhashed-content}

Les seules ressources qui ont un hash dans le manifeste `ngsw.json` sont celles qui étaient présentes dans le répertoire `dist` au moment où le manifeste a été généré.
Les autres ressources, en particulier celles chargées depuis des CDN, ont un contenu inconnu au moment du build ou sont mises à jour plus souvent que l'application n'est déployée.

Si le service worker d'Angular ne dispose pas d'un hash pour vérifier qu'une ressource est valide, il met tout de même son contenu en cache. Dans le même temps, il respecte les en-têtes de mise en cache HTTP en appliquant une politique _stale while revalidate_.
Le service worker d'Angular continue de servir une ressource même après que ses en-têtes de mise en cache HTTP indiquent
qu'elle n'est plus valide. Dans le même temps, il tente d'actualiser la ressource expirée en arrière-plan.
Ainsi, les ressources non hashées défectueuses ne restent pas dans le cache au-delà de leur durée de vie configurée.

### Onglets de l'application {#application-tabs}

Il peut être problématique pour une application que la version des ressources qu'elle reçoit change brusquement ou sans prévenir.
Consultez la section [Versions de l'application](#application-versions) pour une description de ces problèmes.

Le service worker d'Angular fournit une garantie : une application en cours d'exécution continue d'exécuter la même version de l'application.
Si une autre instance de l'application est ouverte dans un nouvel onglet du navigateur, c'est la version la plus récente de l'application qui est servie.
Par conséquent, ce nouvel onglet peut exécuter une version de l'application différente de celle de l'onglet d'origine.

IMPORTANT: Cette garantie est **plus forte** que celle du modèle de déploiement web classique. Sans service worker, rien ne garantit que le code chargé en lazy loading provienne de la même version que le code initial de l'application.

Le service worker d'Angular peut changer la version d'une application en cours d'exécution en cas d'erreur, par exemple :

- La version actuelle devient non valide à cause d'un hash en échec.
- Une erreur sans rapport fait passer le service worker en mode sans échec et le désactive temporairement.

Le service worker d'Angular nettoie les versions de l'application lorsqu'aucun onglet ne les utilise.

Les autres raisons pour lesquelles le service worker d'Angular peut changer la version d'une application en cours d'exécution sont des événements normaux :

- La page est rechargée ou actualisée.
- La page demande l'activation immédiate d'une mise à jour à l'aide du service `SwUpdate`.

### Mises à jour du service worker {#service-worker-updates}

Le service worker d'Angular est un petit script qui s'exécute dans les navigateurs web.
De temps en temps, le service worker est mis à jour avec des corrections de bugs et des améliorations de fonctionnalités.

Le service worker d'Angular est téléchargé lors de la première ouverture de l'application et lorsque l'application est consultée après une période d'inactivité.
Si le service worker change, il est mis à jour en arrière-plan.

La plupart des mises à jour du service worker d'Angular sont transparentes pour l'application. Les anciens caches restent valides et le contenu continue d'être servi normalement.
Il arrive qu'une correction de bug ou une fonctionnalité du service worker d'Angular nécessite l'invalidation des anciens caches.
Dans ce cas, le service worker actualise l'application depuis le réseau de manière transparente.

#### Mettre à jour le service worker lorsque seuls ses en-têtes de réponse changent {#updating-the-service-worker-when-only-its-response-headers-change}

Les navigateurs n'installent un nouveau service worker que lorsque le script du service worker diffère, octet par octet, de celui qui est installé.
Comme `ngsw-worker.js` est généralement identique d'un build à l'autre, modifier uniquement les en-têtes de réponse que le serveur envoie avec lui, comme un `Content-Security-Policy`, ne met pas à jour le service worker installé.
Le service worker continue de s'exécuter avec les en-têtes avec lesquels il a été installé.

Pour que les navigateurs installent à nouveau le service worker, enregistrez-le avec une URL de script versionnée et changez la version chaque fois que ces en-têtes changent :

```ts
provideServiceWorker('ngsw-worker.js?v=2', {
  enabled: !isDevMode(),
  registrationStrategy: 'registerWhenStable:30000',
});
```

Enregistrer une URL de script différente amène le navigateur à récupérer et installer à nouveau le service worker, même si son contenu n'a pas changé.
Le service worker d'Angular résout ses caches et `ngsw.json` par rapport à sa portée d'enregistrement, et non à l'URL de son script ; le query parameter n'a donc aucun effet sur le contenu en cache.

### Contourner le service worker {#bypassing-the-service-worker}

Dans certains cas, vous pouvez vouloir contourner entièrement le service worker et laisser le navigateur gérer la requête.
C'est le cas par exemple lorsque vous dépendez d'une fonctionnalité qui n'est pas encore prise en charge dans les service workers, comme [le suivi de la progression des fichiers envoyés](https://github.com/w3c/ServiceWorker/issues/1141).

Pour contourner le service worker, définissez `ngsw-bypass` comme en-tête de requête ou comme query parameter.
La valeur de l'en-tête ou du query parameter est ignorée et peut être vide ou omise.

### Requêtes du service worker lorsque le serveur est injoignable {#service-worker-requests-when-the-server-cant-be-reached}

Le service worker traite toutes les requêtes, sauf si le [service worker est explicitement contourné](#bypassing-the-service-worker).
Le service worker renvoie soit une réponse en cache, soit envoie la requête au serveur, selon l'état et la configuration du cache.
Le service worker ne met en cache que les réponses aux requêtes qui ne modifient pas de données, comme `GET` et `HEAD`.

Si le service worker reçoit une erreur du serveur ou ne reçoit pas de réponse, il renvoie un statut d'erreur qui indique le résultat de l'appel.
Par exemple, si le service worker ne reçoit pas de réponse, il crée un statut [504 Gateway Timeout](https://developer.mozilla.org/docs/Web/HTTP/Status/504) à renvoyer. Le statut `504` de cet exemple peut être renvoyé parce que le serveur est hors ligne ou que le client est déconnecté.

## Déboguer le service worker d'Angular {#debugging-the-angular-service-worker}

Il peut parfois être nécessaire d'examiner le service worker d'Angular en cours d'exécution pour étudier des problèmes ou vérifier qu'il fonctionne comme prévu.
Les navigateurs fournissent des outils intégrés pour déboguer les service workers, et le service worker d'Angular inclut lui-même des fonctionnalités de débogage utiles.

### Trouver et analyser les informations de débogage {#locating-and-analyzing-debugging-information}

Le service worker d'Angular expose des informations de débogage sous le répertoire virtuel `ngsw/`.
Actuellement, la seule URL exposée est `ngsw/state`.
Voici un exemple du contenu de cette page de débogage :

```shell {hideCopy}

NGSW Debug Info:

Driver version: 13.3.7
Driver state: NORMAL ((nominal))
Latest manifest hash: eea7f5f464f90789b621170af5a569d6be077e5c
Last update check: never

=== Version eea7f5f464f90789b621170af5a569d6be077e5c ===

Clients: 7b79a015-69af-4d3d-9ae6-95ba90c79486, 5bc08295-aaf2-42f3-a4cc-9e4ef9100f65

=== Idle Task Queue ===
Last update tick: 1s496u
Last update run: never
Task queue:

- init post-load (update, cleanup)

Debug log:

```

#### État du driver {#driver-state}

La première ligne indique l'état du driver :

```shell {hideCopy}

Driver state: NORMAL ((nominal))

```

`NORMAL` indique que le service worker fonctionne normalement et n'est pas dans un état dégradé.

Il existe deux états dégradés possibles :

| États dégradés          | Détails                                                                                                                                                                                                                                                                                                                                                                                                                       |
| :---------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `EXISTING_CLIENTS_ONLY` | Le service worker ne dispose pas d'une copie intacte de la dernière version connue de l'application. Les anciennes versions en cache peuvent être utilisées sans risque : les onglets existants continuent donc de s'exécuter depuis le cache, mais les nouveaux chargements de l'application seront servis depuis le réseau. Le service worker tentera de sortir de cet état lorsqu'une nouvelle version de l'application sera détectée et installée, c'est-à-dire lorsqu'un nouveau `ngsw.json` sera disponible. |
| `SAFE_MODE`             | Le service worker ne peut pas garantir que l'utilisation des données en cache est sûre. Soit une erreur inattendue s'est produite, soit toutes les versions en cache sont invalides. Tout le trafic sera servi depuis le réseau, en exécutant le moins de code du service worker possible.                                                                                                                                                                                  |

Dans les deux cas, l'annotation entre parenthèses indique
l'erreur qui a fait passer le service worker dans l'état dégradé.

Ces deux états sont temporaires ; ils ne sont conservés que pendant la durée de vie de l'[instance ServiceWorker](https://developer.mozilla.org/docs/Web/API/ServiceWorkerGlobalScope).
Le navigateur arrête parfois un service worker inactif pour économiser la mémoire et la puissance du processeur, et crée une nouvelle instance de service worker en réponse aux événements réseau.
La nouvelle instance démarre en mode `NORMAL`, quel que soit l'état de l'instance précédente.

#### Hash du dernier manifeste {#latest-manifest-hash}

```shell {hideCopy}

Latest manifest hash: eea7f5f464f90789b621170af5a569d6be077e5c

```

Il s'agit du hash SHA1 de la version la plus récente de l'application connue du service worker.

#### Dernière vérification de mise à jour {#last-update-check}

```shell {hideCopy}

Last update check: never

```

Cette ligne indique la dernière fois que le service worker a vérifié l'existence d'une nouvelle version, ou mise à jour, de l'application.
`never` indique que le service worker n'a jamais vérifié de mise à jour.

Dans ce fichier de débogage d'exemple, la vérification de mise à jour est actuellement planifiée, comme l'explique la section suivante.

#### Version {#version}

```shell {hideCopy}

=== Version eea7f5f464f90789b621170af5a569d6be077e5c ===

Clients: 7b79a015-69af-4d3d-9ae6-95ba90c79486, 5bc08295-aaf2-42f3-a4cc-9e4ef9100f65

```

Dans cet exemple, le service worker a une seule version de l'application en cache, utilisée pour servir deux onglets différents.

HELPFUL: Ce hash de version est le « latest manifest hash » indiqué plus haut. Les deux clients sont sur la dernière version. Chaque client est identifié par son ID issu de l'API `Clients` du navigateur.

#### File des tâches en attente {#idle-task-queue}

```shell {hideCopy}

=== Idle Task Queue ===
Last update tick: 1s496u
Last update run: never
Task queue:

- init post-load (update, cleanup)

```

L'Idle Task Queue est la file de toutes les tâches en attente qui s'exécutent en arrière-plan dans le service worker.
S'il y a des tâches dans la file, elles sont listées avec une description.
Dans cet exemple, le service worker a une seule tâche de ce type planifiée : une opération post-initialisation comprenant une vérification de mise à jour et le nettoyage des caches périmés.

Les compteurs « last update tick/run » donnent le temps écoulé depuis certains événements liés à la file des tâches en attente.
Le compteur « Last update run » indique la dernière fois que les tâches en attente ont réellement été exécutées.
« Last update tick » indique le temps écoulé depuis le dernier événement après lequel la file a pu être traitée.

#### Journal de débogage {#debug-log}

```shell {hideCopy}

Debug log:

```

Les erreurs qui se produisent dans le service worker sont journalisées ici.

### Outils de développement {#developer-tools}

Les navigateurs comme Chrome fournissent des outils de développement pour interagir avec les service workers.
Ces outils peuvent être puissants lorsqu'ils sont bien utilisés, mais il faut garder quelques points en tête.

- Lorsque vous utilisez les outils de développement, le service worker reste actif en arrière-plan et ne redémarre jamais.
  Le comportement observé avec les Dev Tools ouverts peut donc différer de celui que connaîtrait un utilisateur.

- Si vous regardez dans la visionneuse Cache Storage, le cache est souvent obsolète.
  Faites un clic droit sur le titre Cache Storage et actualisez les caches.

- Arrêter puis démarrer le service worker dans le panneau Service Worker déclenche une vérification des mises à jour

## Sécurité du service worker {#service-worker-safety}

Des bugs ou des configurations défectueuses peuvent amener le service worker d'Angular à se comporter de manière inattendue.
Dans ce cas, le service worker d'Angular contient plusieurs mécanismes de sécurité au cas où un administrateur devrait désactiver rapidement le service worker.

### Mécanisme de sécurité {#fail-safe}

Pour désactiver le service worker, renommez le fichier `ngsw.json` ou supprimez-le.
Lorsque la requête du service worker pour `ngsw.json` renvoie une `404`, le service worker supprime tous ses caches et se désenregistre, ce qui revient à s'autodétruire.

### Safety worker {#safety-worker}

<!-- vale Angular.Google_Acronyms = NO -->

Un petit script, `safety-worker.js`, est également inclus dans le package npm `@angular/service-worker`.
Une fois chargé, il se désenregistre du navigateur et supprime les caches du service worker.
Ce script peut être utilisé en dernier recours pour se débarrasser de service workers indésirables déjà installés sur les pages des clients.

<!-- vale Angular.Google_Acronyms = YES -->

CRITICAL: Vous ne pouvez pas enregistrer ce worker directement, car les anciens clients avec un état en cache risquent de ne pas voir le nouvel `index.html` qui installe l'autre script de worker.

Vous devez plutôt servir le contenu de `safety-worker.js` à l'URL du script de service worker que vous cherchez à désenregistrer. Vous devez continuer à le faire jusqu'à être certain que tous les utilisateurs ont bien désenregistré l'ancien worker.
Pour la plupart des sites, cela signifie que vous devez servir le safety worker à l'ancienne URL du service worker indéfiniment.
Ce script peut être utilisé pour désactiver `@angular/service-worker` et supprimer les caches correspondants. Il supprime également tous les autres service workers qui ont pu être servis par le passé sur votre site.

### Changer l'emplacement de votre application {#changing-your-applications-location}

IMPORTANT: Les service workers ne fonctionnent pas derrière une redirection.
Vous avez peut-être déjà rencontré l'erreur `The script resource is behind a redirect, which is disallowed`.

Cela peut poser problème si vous devez changer l'emplacement de votre application.
Si vous mettez en place une redirection de l'ancien emplacement, comme `example.com`, vers le nouveau, `www.example.com` dans cet exemple, le worker cesse de fonctionner.
De plus, la redirection ne se déclenchera même pas pour les utilisateurs qui chargent le site entièrement depuis le service worker.
L'ancien worker, enregistré sur `example.com`, tente de se mettre à jour et envoie une requête à l'ancien emplacement `example.com`. Cette requête est redirigée vers le nouvel emplacement `www.example.com` et provoque l'erreur : `The script resource is behind a redirect, which is disallowed`.

Pour y remédier, vous devrez peut-être désactiver l'ancien worker à l'aide de l'une des techniques précédentes : [mécanisme de sécurité](#fail-safe) ou [safety worker](#safety-worker).

## En savoir plus sur les service workers Angular {#more-on-angular-service-workers}

Les pages suivantes peuvent également vous intéresser :

<docs-pill-row>
  <docs-pill href="ecosystem/service-workers/config" title="Fichier de configuration"/>
  <docs-pill href="ecosystem/service-workers/communications" title="Communiquer avec le service worker"/>
</docs-pill-row>
