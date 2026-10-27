<!-- source: cd6f239e252b49a8e987e2b5e239a2eb7a551cd8 -->
# Migration vers des routes en lazy loading

Ce schematic aide les développeurs à convertir les routes de composants chargées immédiatement en routes chargées en lazy loading. Le processus de build peut ainsi découper le bundle de production en chunks plus petits, pour éviter un gros bundle JS qui inclut toutes les routes, ce qui pénalise le chargement initial de la page d'une application.

Lancez le schematic avec la commande suivante :

```shell
ng generate @angular/core:route-lazy-loading
```

### Option de configuration `path` {#path-config-option}

Par défaut, la migration parcourt toute l'application. Si vous voulez appliquer cette migration à une partie des fichiers, vous pouvez passer l'argument path comme ci-dessous :

```shell
ng generate @angular/core:route-lazy-loading --path src/app/sub-component
```

La valeur du paramètre path est un chemin relatif au sein du projet.

### Comment ça fonctionne ? {#how-does-it-work}

Le schematic tente de trouver tous les endroits où les routes de l'application sont définies :

- `RouterModule.forRoot` et `RouterModule.forChild`
- `Router.resetConfig`
- `provideRouter`
- les variables de type `Routes` ou `Route[]` (par ex. `const routes: Routes = [{...}]`)

La migration vérifie tous les composants des routes, regarde s'ils sont standalone et chargés immédiatement et, si c'est le cas, les convertit en routes chargées en lazy loading.

#### Avant {#before}

```typescript {header: "app.module.ts"}
import {NgModule} from '@angular/core';
import {RouterModule} from '@angular/router';
import {Home} from './home';

@NgModule({
  imports: [
    RouterModule.forRoot([
      {
        path: 'home',
        // Home is standalone and eagerly loaded
        component: Home,
      },
    ]),
  ],
})
export class AppModule {}
```

#### Après {#after}

```typescript {header: "app.module.ts"}
@NgModule({
  imports: [
    RouterModule.forRoot([
      {
        path: 'home',
        // ↓ Home is now lazy loaded
        loadComponent: () => import('./home').then((m) => m.Home),
      },
    ]),
  ],
})
export class AppModule {}
```

Cette migration collecte aussi des informations sur tous les composants déclarés dans des NgModules et affiche la liste des routes qui les utilisent (avec l'emplacement du fichier correspondant). Envisagez de rendre ces composants standalone et relancez cette migration. Vous pouvez utiliser une migration existante ([voir](reference/migrations/standalone)) pour convertir ces composants en standalone.
