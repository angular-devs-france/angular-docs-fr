<!-- source: 320215d5afb9b2199aa6d72bb98e19d76249401c -->
<docs-decorative-header title="Le routing dans Angular" imgSrc="adev/src/assets/images/routing.svg"> <!-- markdownlint-disable-line -->
Le routing vous aide à changer ce que l'utilisateur voit dans une application monopage.
</docs-decorative-header>

Angular Router (`@angular/router`) est la library officielle pour gérer la navigation dans les applications Angular et une partie centrale du framework. Elle est incluse par défaut dans tous les projets créés avec Angular CLI.

## Pourquoi le routing est-il nécessaire dans une SPA ? {#why-is-routing-necessary-in-a-spa}

Lorsque vous naviguez vers une URL dans votre navigateur web, le navigateur effectue normalement une requête réseau vers un serveur web et affiche la page HTML renvoyée. Lorsque vous naviguez vers une autre URL, par exemple en cliquant sur un lien, le navigateur effectue une nouvelle requête réseau et remplace toute la page par une nouvelle.

Une application monopage (SPA) se distingue en ce que le navigateur n'effectue une requête vers un serveur web que pour la première page, `index.html`. Ensuite, un router côté client prend le relais et contrôle le contenu affiché en fonction de l'URL. Lorsqu'un utilisateur navigue vers une autre URL, le router met à jour le contenu de la page sur place, sans déclencher de rechargement complet de la page.

## Comment Angular gère le routing {#how-angular-manages-routing}

Le routing dans Angular se compose de trois parties principales :

1. Les **routes** définissent quel composant s'affiche lorsqu'un utilisateur visite une URL donnée.
2. Les **outlets** sont des placeholders dans vos templates qui chargent et affichent dynamiquement des composants en fonction de la route active.
3. Les **liens** permettent aux utilisateurs de naviguer entre les différentes routes de votre application sans déclencher de rechargement complet de la page.

En outre, la library de routing d'Angular offre des fonctionnalités supplémentaires, comme :

- Les routes imbriquées
- La navigation programmatique
- Les paramètres de route, les query parameters et les routes wildcard
- Les informations sur la route activée avec `ActivatedRoute`
- La récupération de données avec les resources de route et les resolvers
- Les effets de View Transitions
- Les guards de navigation

## Étapes suivantes {#next-steps}

Découvrez comment [définir des routes avec le router d'Angular](/guide/routing/define-routes).
