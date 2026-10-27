<!-- source: bcd7908ca3ce51b0a3c8b1768ee3913754329506 -->
# Bien démarrer avec les service workers

Ce document explique comment activer la prise en charge du service worker Angular dans les projets créés avec l'[Angular CLI](tools/cli). Il utilise ensuite un exemple pour vous montrer un service worker en action, avec le chargement et la mise en cache de base.

## Ajouter un service worker à votre projet {#adding-a-service-worker-to-your-project}

Pour configurer le service worker Angular dans votre projet, lancez la commande CLI suivante :

```shell

ng add @angular/pwa

```

La CLI configure votre application pour utiliser les service workers en effectuant les actions suivantes :

1. Ajoute le package `@angular/service-worker` à votre projet.
1. Active la prise en charge du service worker dans le build de la CLI.
1. Importe et enregistre le service worker dans les providers racine de l'application.
1. Met à jour le fichier `index.html` :
   - Ajoute un lien vers le fichier `manifest.webmanifest`
   - Ajoute une balise meta pour `theme-color`
1. Installe des fichiers d'icônes pour la Progressive Web App (PWA) installée.
1. Crée le fichier de configuration du service worker, appelé [`ngsw-config.json`](ecosystem/service-workers/config),
   qui définit les comportements de mise en cache et d'autres paramètres.

Compilez maintenant le projet :

```shell

ng build

```

Le projet CLI est maintenant configuré pour utiliser le service worker Angular.

## Le service worker en action : une visite guidée {#service-worker-in-action-a-tour}

Cette section montre un service worker en action,
à l'aide d'une application d'exemple. Pour activer la prise en charge du service worker pendant le développement local, utilisez la configuration de production avec la commande suivante :

```shell

ng serve --configuration=production

```

Vous pouvez aussi utiliser le [`http-server package`](https://www.npmjs.com/package/http-server) de
npm, qui prend en charge les applications avec service worker. Lancez-le sans installation avec :

```shell

npx http-server -p 8080 -c-1 dist/<project-name>/browser

```

Cela sert votre application avec la prise en charge du service worker sur http://localhost:8080.

### Chargement initial {#initial-load}

Avec le serveur lancé sur le port `8080`, ouvrez `http://localhost:8080` dans votre navigateur.
Votre application devrait se charger normalement.

TIP: Lorsque vous testez des service workers Angular, il est judicieux d'utiliser une fenêtre de navigation privée afin que le service worker ne finisse pas par lire un état résiduel antérieur, ce qui peut provoquer un comportement inattendu.

HELPFUL: Si vous n'utilisez pas HTTPS, le service worker ne sera enregistré que lorsque vous accédez à l'application sur `localhost`.

### Simuler un problème réseau {#simulating-a-network-issue}

Pour simuler un problème réseau, désactivez les échanges réseau de votre application.

Dans Chrome :

1. Sélectionnez **Tools** > **Developer Tools** (depuis le menu Chrome situé dans le coin supérieur droit).
1. Allez dans l'onglet **Network**.
1. Sélectionnez **Offline** dans le menu déroulant **Throttling**.

<img alt="L'option Offline est sélectionnée dans l'onglet Network" src="assets/images/guide/service-worker/offline-option.png">

L'application n'a maintenant plus accès au réseau.

Pour les applications qui n'utilisent pas le service worker Angular, un rafraîchissement afficherait maintenant la page de Chrome signalant l'absence de connexion Internet, avec le message « There is no Internet connection ».

Avec l'ajout d'un service worker Angular, le comportement de l'application change.
Lors d'un rafraîchissement, la page se charge normalement.

Regardez l'onglet Network pour vérifier que le service worker est actif.

<img alt="Les requêtes sont marquées comme provenant du ServiceWorker" src="assets/images/guide/service-worker/sw-active.png">

HELPFUL: Dans la colonne « Size », l'état des requêtes est `(ServiceWorker)`.
Cela signifie que les ressources ne sont pas chargées depuis le réseau.
Elles sont chargées depuis le cache du service worker.

### Qu'est-ce qui est mis en cache ? {#whats-being-cached}

Remarquez que tous les fichiers dont le navigateur a besoin pour afficher cette application sont en cache.
La configuration de base de `ngsw-config.json` est prévue pour mettre en cache les ressources spécifiques utilisées par la CLI :

- `index.html`
- `favicon.ico`
- Les artefacts de build (bundles JS et CSS)
- Tout ce qui se trouve sous `assets`
- Les images et polices situées directement sous l'`outputPath` configuré (par défaut `./dist/<project-name>/`) ou sous `resourcesOutputPath`.
  Consultez la documentation de `ng build` pour plus d'informations sur ces options.

IMPORTANT: Le fichier `ngsw-config.json` généré inclut une liste limitée d'extensions de polices et d'images pouvant être mises en cache. Dans certains cas, vous voudrez peut-être modifier le pattern glob selon vos besoins.

IMPORTANT: Si les chemins `resourcesOutputPath` ou `assets` sont modifiés après la génération du fichier de configuration, vous devez modifier les chemins manuellement dans `ngsw-config.json`.

### Modifier votre application {#making-changes-to-your-application}

Maintenant que vous avez vu comment les service workers mettent votre application en cache, l'étape suivante consiste à comprendre le fonctionnement des mises à jour.
Modifiez l'application et observez le service worker installer la mise à jour :

1. Si vous testez dans une fenêtre de navigation privée, ouvrez un second onglet vide.
   Cela maintient la session de navigation privée et l'état du cache pendant votre test.

1. Fermez l'onglet de l'application, mais pas la fenêtre.
   Cela devrait aussi fermer les Developer Tools.

1. Arrêtez `http-server` (Ctrl-c).
1. Ouvrez `src/app/app.component.html` pour le modifier.
1. Remplacez le texte `Welcome to {{title}}!` par `Bienvenue à {{title}}!`.
1. Compilez et relancez le serveur :

```shell
    ng build
    npx http-server -p 8080 -c-1 dist/<project-name>/browser
```

### Mettre à jour votre application dans le navigateur {#updating-your-application-in-the-browser}

Regardez maintenant comment le navigateur et le service worker gèrent l'application mise à jour.

1. Ouvrez de nouveau [http://localhost:8080](http://localhost:8080) dans la même fenêtre.
   Que se passe-t-il ?

   <img alt="Le message affiche toujours Welcome to Service Workers!" src="assets/images/guide/service-worker/welcome-msg-en.png">

   Qu'est-ce qui n'a pas fonctionné ?
   _Rien, en réalité !_
   Le service worker Angular fait son travail et sert la version de l'application qu'il a **installée**, même si une mise à jour est disponible.
   Par souci de rapidité, le service worker n'attend pas de vérifier les mises à jour avant de servir l'application qu'il a mise en cache.

   Consultez les logs de `http-server` pour voir le service worker demander `/ngsw.json`.

   ```text
   [2023-09-07T00:37:24.372Z]  "GET /ngsw.json?ngsw-cache-bust=0.9365263935102124" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36"
   ```

   C'est ainsi que le service worker vérifie les mises à jour.

1. Rafraîchissez la page.

   <img alt="Le texte affiche désormais Bienvenue à app!" src="assets/images/guide/service-worker/welcome-msg-fr.png">

   Le service worker a installé la version mise à jour de votre application _en arrière-plan_, et au prochain chargement ou rechargement de la page, le service worker passe à la dernière version.

## Configuration du service worker {#service-worker-configuration}

Les service workers Angular prennent en charge des options de configuration complètes via l'interface `SwRegistrationOptions`, qui offre un contrôle fin du comportement d'enregistrement, de la mise en cache et de l'exécution des scripts.

### Activer et désactiver les service workers {#enabling-and-disabling-service-workers}

L'option `enabled` détermine si le service worker sera enregistré et si les services associés tenteront de communiquer avec lui.

```ts
import {ApplicationConfig, isDevMode} from '@angular/core';
import {provideServiceWorker} from '@angular/service-worker';

export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(), // Disable in development, enable in production
    }),
  ],
};
```

### Contrôle du cache avec updateViaCache {#cache-control-with-updateviacache}

L'option `updateViaCache` détermine comment le navigateur consulte le cache HTTP lors des mises à jour du service worker. Elle offre un contrôle fin sur le moment où le navigateur récupère les scripts du service worker mis à jour et les modules importés.

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      updateViaCache: 'imports',
    }),
  ],
};
```

L'option `updateViaCache` accepte les valeurs suivantes :

- **`'imports'`** - Le cache HTTP est consulté pour les scripts importés par le script du service worker, mais pas pour le script du service worker lui-même
- **`'all'`** - Le cache HTTP est consulté à la fois pour le script du service worker et pour ses scripts importés
- **`'none'`** - Le cache HTTP n'est consulté ni pour le script du service worker ni pour ses scripts importés

### Prise en charge des modules ES avec l'option type {#es-module-support-with-type-option}

L'option `type` permet de spécifier le type de script lors de l'enregistrement des service workers, ce qui permet d'utiliser les fonctionnalités des modules ES dans vos scripts de service worker.

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      type: 'module', // Enable ES module features
    }),
  ],
};
```

L'option `type` accepte les valeurs suivantes :

- **`'classic'`** (par défaut) - Exécution traditionnelle du script du service worker. Les fonctionnalités des modules ES comme `import` et `export` NE sont PAS autorisées dans le script
- **`'module'`** - Enregistre le script en tant que module ES. Permet d'utiliser la syntaxe `import`/`export` et les fonctionnalités des modules

### Contrôle de la portée d'enregistrement {#registration-scope-control}

L'option `scope` définit la portée d'enregistrement du service worker, qui détermine la plage d'URL qu'il peut contrôler.

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      scope: '/app/', // Service worker will only control URLs under /app/
    }),
  ],
};
```

- Détermine quelles URL le service worker peut intercepter et gérer
- Par défaut, la portée est le répertoire qui contient le script du service worker
- Utilisée lors de l'appel à `ServiceWorkerContainer.register()`

### Configuration de la stratégie d'enregistrement {#registration-strategy-configuration}

L'option `registrationStrategy` définit quand le service worker sera enregistré auprès du navigateur, ce qui permet de contrôler le moment de l'enregistrement.

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerWhenStable:30000',
    }),
  ],
};
```

Stratégies d'enregistrement disponibles :

- **`'registerWhenStable:timeout'`** (par défaut : `'registerWhenStable:30000'`) - Enregistre dès que l'application se stabilise (aucune micro-tâche ni macro-tâche en attente), mais au plus tard après le délai spécifié en millisecondes
  Le délai est obligatoire. Sans lui, `'registerWhenStable'` enregistre le service worker immédiatement.
- **`'registerImmediately'`** - Enregistre le service worker immédiatement
- **`'registerWithDelay:timeout'`** - Enregistre après le délai spécifié en millisecondes

```ts
// Register immediately
export const immediateConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerImmediately',
    }),
  ],
};

// Register with a 5-second delay
export const delayedConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerWithDelay:5000',
    }),
  ],
};
```

Vous pouvez aussi fournir une fonction fabrique renvoyant un Observable pour personnaliser le moment de l'enregistrement :

```ts
import {ApplicationConfig, isDevMode} from '@angular/core';
import {provideServiceWorker} from '@angular/service-worker';
import {timer} from 'rxjs';

export const customConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('ngsw-worker.js', {
      enabled: !isDevMode(),
      registrationStrategy: () => timer(10_000), // Register after 10 seconds
    }),
  ],
};
```

## En savoir plus sur les service workers Angular {#more-on-angular-service-workers}

Ces sujets pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="ecosystem/service-workers/config" title="Fichier de configuration"/>
  <docs-pill href="ecosystem/service-workers/communications" title="Communiquer avec le service worker"/>
  <docs-pill href="ecosystem/service-workers/push-notifications" title="Notifications push"/>
  <docs-pill href="ecosystem/service-workers/devops" title="DevOps du service worker"/>
  <docs-pill href="ecosystem/service-workers/app-shell" title="Pattern app shell"/>
</docs-pill-row>
