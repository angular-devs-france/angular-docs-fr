<!-- source: 2a5dbaeac9f6b5dc005bc1a5d8797e2d8587d201 -->
# Scripts de service worker personnalisés

Bien que le service worker Angular offre d'excellentes capacités, vous pouvez avoir besoin d'ajouter des fonctionnalités personnalisées, comme la gestion des notifications push, la synchronisation en arrière-plan ou d'autres événements de service worker. Vous pouvez créer un script de service worker personnalisé qui importe et étend le service worker Angular.

## Créer un service worker personnalisé {#creating-a-custom-service-worker}

Pour créer un service worker personnalisé qui étend les fonctionnalités d'Angular :

1. Créez un fichier de service worker personnalisé (par exemple, `custom-sw.js`) dans votre répertoire `src` :

```js
// Import the Angular service worker
importScripts('./ngsw-worker.js');

(function () {
  'use strict';

  // Add custom notification click handler
  self.addEventListener('notificationclick', (event) => {
    console.log('Custom notification click handler');
    console.log('Notification details:', event.notification);

    // Handle notification click - open URL if provided
    if (clients.openWindow && event.notification.data.url) {
      event.waitUntil(clients.openWindow(event.notification.data.url));
      console.log('Opening URL:', event.notification.data.url);
    }
  });

  // Add custom background sync handler
  self.addEventListener('sync', (event) => {
    console.log('Custom background sync handler');

    if (event.tag === 'background-sync') {
      event.waitUntil(doBackgroundSync());
    }
  });

  function doBackgroundSync() {
    // Implement your background sync logic here
    return fetch('https://example.com/api/sync')
      .then((response) => response.json())
      .then((data) => console.log('Background sync completed:', data))
      .catch((error) => console.error('Background sync failed:', error));
  }
})();
```

2. Modifiez votre fichier `angular.json` pour utiliser le service worker personnalisé :

```json
{
  "projects": {
    "your-app": {
      "architect": {
        "build": {
          "options": {
            "assets": [
              {
                "glob": "**/*",
                "input": "public"
              },
              "app/src/custom-sw.js"
            ]
          }
        }
      }
    }
  }
}
```

3. Configurez l'enregistrement du service worker pour qu'il utilise votre script personnalisé :

```ts
import {ApplicationConfig, isDevMode} from '@angular/core';
import {provideServiceWorker} from '@angular/service-worker';

export const appConfig: ApplicationConfig = {
  providers: [
    provideServiceWorker('custom-sw.js', {
      enabled: !isDevMode(),
      registrationStrategy: 'registerWhenStable:30000',
    }),
  ],
};
```

### Bonnes pratiques pour les service workers personnalisés {#best-practices-for-custom-service-workers}

Lorsque vous étendez le service worker Angular :

- **Importez toujours le service worker Angular en premier** avec `importScripts('./ngsw-worker.js')` pour bénéficier de toutes les fonctionnalités de mise en cache et de mise à jour
- **Encapsulez votre code personnalisé dans une IIFE** (Immediately Invoked Function Expression) pour éviter de polluer la portée globale
- **Utilisez `event.waitUntil()`** pour les opérations asynchrones afin de garantir qu'elles se terminent avant l'arrêt du service worker
- **Testez minutieusement** dans les environnements de développement et de production
- **Gérez les erreurs proprement** pour éviter que votre code personnalisé ne casse les fonctionnalités du service worker Angular

### Cas d'usage courants {#common-use-cases}

Les service workers personnalisés servent couramment pour :

- **Notifications push** : gérer les messages push entrants et afficher des notifications
- **Synchronisation en arrière-plan** : synchroniser les données quand la connexion réseau est rétablie
- **Navigation personnalisée** : gérer des scénarios particuliers de routing ou de page hors ligne
