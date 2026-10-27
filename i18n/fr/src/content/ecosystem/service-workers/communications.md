<!-- source: f10efab1473f5d2d26886635ceb9004f6ec40ee7 -->
# Communiquer avec le service worker

Activer la prise en charge du service worker ne se limite pas à enregistrer le service worker : cela fournit aussi des services que vous pouvez utiliser pour interagir avec le service worker et contrôler la mise en cache de votre application.

## Le service `SwUpdate` {#swupdate-service}

Le service `SwUpdate` vous donne accès à des événements qui indiquent quand le service worker découvre et installe une mise à jour disponible pour votre application.

Le service `SwUpdate` prend en charge trois opérations distinctes :

- Recevoir des notifications quand une version mise à jour est _détectée_ sur le serveur, _installée et prête_ à être utilisée localement, ou quand une _installation échoue_.
- Demander au service worker de vérifier auprès du serveur s'il existe de nouvelles mises à jour.
- Demander au service worker d'activer la dernière version de l'application pour l'onglet courant.

### Mises à jour de version {#version-updates}

`versionUpdates` est une propriété `Observable` de `SwUpdate` qui émet quatre types d'événements :

| Types d'événements               | Détails                                                                                                                                                                     |
| :------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `VersionDetectedEvent`           | Émis quand le service worker a détecté une nouvelle version de l'application sur le serveur et s'apprête à la télécharger.                                                  |
| `NoNewVersionDetectedEvent`      | Émis quand le service worker a vérifié la version de l'application sur le serveur et n'a pas trouvé de nouvelle version.                                                    |
| `VersionReadyEvent`              | Émis quand une nouvelle version de l'application peut être activée par les clients. Il peut servir à signaler une mise à jour disponible à l'utilisateur ou à l'inviter à rafraîchir la page. |
| `VersionInstallationFailedEvent` | Émis quand l'installation d'une nouvelle version a échoué. Il peut servir à des fins de journalisation ou de supervision.                                                   |

<docs-code header="log-update.service.ts" path="adev/src/content/examples/service-worker-getting-started/src/app/log-update.service.ts" region="sw-update"/>

### Vérifier les mises à jour {#checking-for-updates}

Il est possible de demander au service worker de vérifier si des mises à jour ont été déployées sur le serveur.
Le service worker vérifie les mises à jour lors de son initialisation et à chaque requête de navigation, c'est-à-dire quand l'utilisateur arrive sur votre application depuis une autre adresse.
Vous pouvez cependant choisir de vérifier les mises à jour manuellement si votre site change fréquemment ou si vous voulez que les mises à jour aient lieu selon un calendrier.

Faites-le avec la méthode `checkForUpdate()` :

<docs-code header="check-for-update.service.ts" path="adev/src/content/examples/service-worker-getting-started/src/app/check-for-update.service.ts"/>

Cette méthode renvoie une `Promise<boolean>` qui indique si une mise à jour est disponible pour activation.
La vérification peut échouer, ce qui provoque le rejet de la `Promise`.

<docs-callout important title="Stabilisation et enregistrement du service worker">
Pour éviter de pénaliser le rendu initial de la page, le service du service worker Angular attend par défaut jusqu'à 30 secondes que l'application se stabilise avant d'enregistrer le script du ServiceWorker.

Interroger en permanence le serveur pour détecter des mises à jour, par exemple avec [setInterval()](https://developer.mozilla.org/docs/Web/API/WindowOrWorkerGlobalScope/setInterval) ou [interval()](https://rxjs.dev/api/index/function/interval) de RxJS, empêche l'application de se stabiliser, et le script du ServiceWorker n'est pas enregistré auprès du navigateur avant que la limite de 30 secondes soit atteinte.

C'est vrai pour tout type d'interrogation périodique effectuée par votre application.
Consultez la documentation de [isStable](api/core/ApplicationRef#isStable) pour plus d'informations.

Évitez ce délai en attendant d'abord que l'application se stabilise avant de commencer à interroger le serveur pour détecter des mises à jour, comme le montre l'exemple précédent.
Vous pouvez aussi définir une autre [stratégie d'enregistrement](api/service-worker/SwRegistrationOptions#registrationStrategy) pour le ServiceWorker.
</docs-callout>

### Passer à la dernière version {#updating-to-the-latest-version}

Vous pouvez mettre à jour un onglet existant vers la dernière version en rechargeant la page dès qu'une nouvelle version est prête.
Pour ne pas interrompre l'utilisateur dans ce qu'il fait, il est généralement préférable de lui demander de confirmer qu'il accepte de recharger la page et de passer à la dernière version :

<docs-code header="prompt-update.service.ts" path="adev/src/content/examples/service-worker-getting-started/src/app/prompt-update.service.ts" region="sw-version-ready"/>

<docs-callout important title="Risques d'une mise à jour sans rechargement">
Appeler `activateUpdate()` met à jour un onglet vers la dernière version sans recharger la page, mais cela peut casser l'application.

Mettre à jour sans recharger peut créer une incohérence de version entre le shell de l'application et les autres ressources de la page, comme les chunks chargés en lazy loading, dont les noms de fichiers peuvent changer d'une version à l'autre.

N'utilisez `activateUpdate()` que si vous êtes certain que c'est sans risque pour votre cas d'usage.
</docs-callout>

### Gérer un état irrécupérable {#handling-an-unrecoverable-state}

Dans certains cas, la version de l'application utilisée par le service worker pour servir un client peut se trouver dans un état cassé, impossible à rétablir sans un rechargement complet de la page.

Par exemple, imaginez le scénario suivant :

1. Un utilisateur ouvre l'application pour la première fois et le service worker met en cache la dernière version de l'application.
   Supposons que les ressources en cache de l'application incluent `index.html`, `main.<main-hash-1>.js` et `lazy-chunk.<lazy-hash-1>.js`.

1. L'utilisateur ferme l'application et ne l'ouvre plus pendant un moment.
1. Quelque temps plus tard, une nouvelle version de l'application est déployée sur le serveur.
   Cette nouvelle version inclut les fichiers `index.html`, `main.<main-hash-2>.js` et `lazy-chunk.<lazy-hash-2>.js`.

IMPORTANT: Les hashes sont maintenant différents, parce que le contenu des fichiers a changé. L'ancienne version n'est plus disponible sur le serveur.

1. Entre-temps, le navigateur de l'utilisateur décide d'évincer `lazy-chunk.<lazy-hash-1>.js` de son cache.
   Les navigateurs peuvent décider d'évincer certaines ressources (ou toutes) d'un cache afin de récupérer de l'espace disque.

1. L'utilisateur ouvre de nouveau l'application.
   Le service worker sert la dernière version qu'il connaît à ce moment-là, à savoir l'ancienne version (`index.html` et `main.<main-hash-1>.js`).

1. Plus tard, l'application demande le bundle chargé en lazy loading, `lazy-chunk.<lazy-hash-1>.js`.
1. Le service worker ne parvient pas à trouver la ressource dans le cache (rappelez-vous que le navigateur l'a évincée).
   Il ne peut pas non plus la récupérer depuis le serveur (parce que le serveur n'a plus que `lazy-chunk.<lazy-hash-2>.js`, issu de la nouvelle version).

Dans le scénario précédent, le service worker ne peut pas servir une ressource qui serait normalement en cache.
Cette version précise de l'application est cassée et il n'existe aucun moyen de rétablir l'état du client sans recharger la page.
Dans de tels cas, le service worker notifie le client en envoyant un événement `UnrecoverableStateEvent`.
Abonnez-vous à `SwUpdate#unrecoverable` pour être notifié et gérer ces erreurs.

<docs-code header="handle-unrecoverable-state.service.ts" path="adev/src/content/examples/service-worker-getting-started/src/app/handle-unrecoverable-state.service.ts" region="sw-unrecoverable-state"/>

## En savoir plus sur les service workers Angular {#more-on-angular-service-workers}

Ces sujets pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="ecosystem/service-workers/push-notifications" title="Notifications push"/>
  <docs-pill href="ecosystem/service-workers/devops" title="DevOps du service worker"/>
</docs-pill-row>
