<!-- source: 621968d05ad8937276e5fc4a1189d4fe9e2be913 -->
# Stratégies de rendu dans Angular

Ce guide vous aide à choisir la bonne stratégie de rendu pour les différentes parties de votre application Angular.

## Que sont les stratégies de rendu ? {#what-are-rendering-strategies}

Les stratégies de rendu déterminent quand et où le contenu HTML de votre application Angular est généré. Chaque stratégie offre des compromis différents entre les performances du chargement initial de la page, l'interactivité, les capacités SEO et l'utilisation des ressources serveur.

Angular prend en charge trois stratégies de rendu principales :

- **Rendu côté client (CSR)** - Le contenu est rendu entièrement dans le navigateur
- **Génération de site statique (SSG/prérendu)** - Le contenu est prérendu au moment du build
- **Rendu côté serveur (SSR)** - Le contenu est rendu sur le serveur lors de la requête initiale d'une route

## Rendu côté client (CSR) {#client-side-rendering-csr}

**Le CSR est le comportement par défaut d'Angular.** Le contenu est rendu entièrement dans le navigateur une fois le JavaScript chargé.

### Quand utiliser le CSR {#when-to-use-csr}

✅ Il peut être un bon choix pour :

- Les applications interactives (tableaux de bord, panneaux d'administration)
- Les applications temps réel
- Les outils internes pour lesquels le SEO n'a pas d'importance
- Les single-page applications avec un état côté client complexe

❌ Dans la mesure du possible, évitez-le pour :

- Le contenu public qui a besoin de SEO
- Les pages où les performances du chargement initial sont critiques

### Compromis du CSR {#csr-trade-offs}

| Aspect            | Impact                                                   |
| :---------------- | :------------------------------------------------------- |
| **SEO**           | Faible - contenu invisible pour les robots d'indexation tant que le JS ne s'est pas exécuté |
| **Chargement initial**  | Plus lent - le JavaScript doit d'abord être téléchargé et exécuté      |
| **Interactivité** | Immédiate une fois chargé                                    |
| **Besoins serveur**  | Minimes, hormis un peu de configuration                    |
| **Complexité**    | La plus simple, car fonctionne avec une configuration minimale     |

## Génération de site statique (SSG/prérendu) {#static-site-generation-ssg-prerendering}

**Le SSG prérend les pages au moment du build** sous forme de fichiers HTML statiques. Le serveur envoie du HTML pré-construit pour le chargement initial de la page. Après l'hydratation, votre application s'exécute entièrement dans le navigateur comme une SPA classique : les navigations suivantes, les changements de route et les appels d'API se font tous côté client, sans rendu serveur.

### Quand utiliser le SSG {#when-to-use-ssg}

✅ Il peut être un bon choix pour :

- Les pages marketing et les landing pages
- Les articles de blog et la documentation
- Les catalogues de produits au contenu stable
- Le contenu qui ne varie pas selon l'utilisateur

❌ Dans la mesure du possible, évitez-le pour :

- Le contenu propre à chaque utilisateur
- Les données qui changent fréquemment
- Les informations en temps réel

### Compromis du SSG {#ssg-trade-offs}

| Aspect              | Impact                                      |
| :------------------ | :------------------------------------------ |
| **SEO**             | Excellent - HTML complet disponible immédiatement |
| **Chargement initial**    | Le plus rapide - HTML pré-généré                |
| **Interactivité**   | Une fois l'hydratation terminée                   |
| **Besoins serveur**    | Aucun pour la diffusion (compatible CDN)             |
| **Temps de build**      | Plus long - toutes les pages sont générées à l'avance        |
| **Mises à jour du contenu** | Nécessitent un nouveau build et un redéploiement               |

📖 **Mise en œuvre :** consultez [Personnaliser le prérendu au moment du build](guide/ssr#customizing-build-time-prerendering-ssg) dans le guide SSR.

## Rendu côté serveur (SSR) {#server-side-rendering-ssr}

**Le SSR génère le HTML sur le serveur lors de la requête initiale d'une route**, ce qui fournit du contenu dynamique avec un bon SEO. Le serveur effectue le rendu HTML et l'envoie au client.

Une fois que le client a affiché la page, Angular [hydrate](/guide/hydration#what-is-hydration) l'application, qui s'exécute ensuite entièrement dans le navigateur comme une SPA classique : les navigations suivantes, les changements de route et les appels d'API se font tous côté client, sans rendu serveur supplémentaire.

### Quand utiliser le SSR {#when-to-use-ssr}

✅ Il peut être un bon choix pour :

- Les pages produit d'e-commerce (prix et stocks dynamiques)
- Les sites d'actualité et les fils de réseaux sociaux
- Le contenu personnalisé qui change fréquemment

❌ Dans la mesure du possible, évitez-le pour :

- Le contenu statique (utilisez plutôt le SSG)
- Les cas où les coûts serveur posent problème

### Compromis du SSR {#ssr-trade-offs}

| Aspect              | Impact                                              |
| :------------------ | :-------------------------------------------------- |
| **SEO**             | Excellent - HTML complet pour les robots d'indexation                  |
| **Chargement initial**    | Rapide - contenu visible immédiatement                 |
| **Interactivité**   | Retardée jusqu'à l'hydratation                             |
| **Besoins serveur**    | Nécessite un serveur                                     |
| **Personnalisation** | Accès complet au contexte utilisateur                         |
| **Coûts serveur**    | Plus élevés - rendu lors de la requête initiale de chaque route |

📖 **Mise en œuvre :** consultez [Routing côté serveur](guide/ssr#server-routing) et [Écrire des composants compatibles avec le serveur](guide/ssr#authoring-server-compatible-components) dans le guide SSR.

## Choisir la bonne stratégie {#choosing-the-right-strategy}

### Matrice de décision {#decision-matrix}

| Si vous avez besoin de...             | Utilisez cette stratégie | Pourquoi                                              |
| :------------------------- | :---------------- | :----------------------------------------------- |
| **SEO + contenu statique**   | SSG               | HTML prérendu, chargement le plus rapide                  |
| **SEO + contenu dynamique**  | SSR               | Contenu à jour lors de la requête initiale d'une route |
| **Pas de SEO + interactivité** | CSR               | Le plus simple, aucun serveur nécessaire                       |
| **Besoins mixtes**     | Hybride            | Stratégies différentes selon la route                   |

## Rendre le SSR/SSG interactif grâce à l'hydratation {#making-ssr-ssg-interactive-with-hydration}

Avec le SSR ou le SSG, Angular « hydrate » le HTML rendu par le serveur pour le rendre interactif.

**Stratégies disponibles :**

- **Hydratation complète** - Toute l'application devient interactive d'un coup (par défaut)
- **Hydratation incrémentale** - Des parties deviennent interactives au besoin (meilleures performances)
- **Event replay** - Capture les clics avant la fin de l'hydratation

📖 **En savoir plus :**

- [Guide de l'hydratation](guide/hydration) - Configuration complète de l'hydratation
- [Hydratation incrémentale](guide/incremental-hydration) - Hydratation avancée avec les blocs `@defer`

## Prochaines étapes {#next-steps}

<docs-pill-row>
  <docs-pill href="/guide/ssr" title="Rendu côté serveur"/>
  <docs-pill href="/guide/hydration" title="Hydratation"/>
  <docs-pill href="/guide/incremental-hydration" title="Hydratation incrémentale"/>
</docs-pill-row>
