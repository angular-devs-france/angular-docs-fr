<!-- source: 29e4b4219f1d7f6349aa0f6c954b72de670657b0 -->
# Servir des applications Angular en développement

Vous pouvez servir votre application Angular CLI avec la commande `ng serve`.
Celle-ci compile votre application, ignore les optimisations inutiles, démarre un serveur de développement, puis recompile et recharge automatiquement la page à chaque modification ultérieure.
Vous pouvez arrêter le serveur en appuyant sur `Ctrl+C`.

`ng serve` exécute uniquement le builder de la cible `serve` du projet par défaut, tel que spécifié dans `angular.json`. Bien que n'importe quel builder puisse être utilisé ici, le builder le plus courant (et celui par défaut) est `@angular/build:dev-server`.

Vous pouvez déterminer quel builder est utilisé pour un projet donné en consultant la cible `serve` de ce projet.

```json
{
  "projects": {
    "my-app": {
      "architect": {
        // `ng serve` invokes the Architect target named `serve`.
        "serve": {
          "builder": "@angular/build:dev-server"
          // ...
        },
        "build": {/* ... */},
        "test": {/* ... */}
      }
    }
  }
}
```

## Utiliser un proxy vers un serveur backend {#proxying-to-a-backend-server}

Utilisez la [prise en charge du proxy](https://vite.dev/config/server-options#server-proxy) pour rediriger certaines URL vers un serveur backend, en passant un fichier à l'option de build `--proxy-config`.
Par exemple, pour rediriger tous les appels vers `http://localhost:4200/api` vers un serveur s'exécutant sur `http://localhost:3000/api`, suivez les étapes ci-dessous.

1. Créez un fichier `proxy.conf.json` dans le dossier `src/` de votre projet.
1. Ajoutez le contenu suivant au nouveau fichier de proxy :

```json
{
  "/api/**": {
    "target": "http://localhost:3000",
    "secure": false
  }
}
```

1. Dans le fichier de configuration de la CLI, `angular.json`, ajoutez l'option `proxyConfig` à la cible `serve` :

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "serve": {
          "builder": "@angular/build:dev-server",
          "options": {
            "proxyConfig": "src/proxy.conf.json"
          }
        }
      }
    }
  }
}
```

1. Pour lancer le serveur de développement avec cette configuration de proxy, exécutez `ng serve`.

NOTE: Pour appliquer les modifications apportées à votre fichier de configuration du proxy, vous devez redémarrer le processus `ng serve`.

### Comportement de la correspondance des chemins {#path-matching-behavior}

- `/api` correspond à `/api` et à tout chemin qui commence par celui-ci, comme `/api/users/123` ou `/api-docs`.
- `/api/*` correspond à `/api/users`, mais pas à `/api` ni à `/api/users/123`.
- `/api/**` correspond à `/api`, `/api/users` et `/api/users/123`, mais pas à `/api-docs`.
