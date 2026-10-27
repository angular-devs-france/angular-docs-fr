<!-- source: 0754c74c34ffce0dfa2a14813cc162f8f6859f98 -->
# Performances

Angular inclut de nombreuses optimisations par défaut, mais à mesure que les applications grandissent, vous aurez peut-être besoin d'affiner à la fois la vitesse de chargement de votre application et sa réactivité à l'usage. Ces guides présentent les outils et techniques qu'Angular fournit pour vous aider à créer des applications rapides.

## Performances de chargement {#loading-performance}

Les performances de chargement déterminent la rapidité avec laquelle votre application devient visible et interactive. Un chargement lent impacte directement les [Core Web Vitals](https://web.dev/vitals/) comme le Largest Contentful Paint (LCP) et le Time to First Byte (TTFB).

| Technique | Ce qu'elle fait | Quand l'utiliser |
| :------------------------------------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------------------------------- |
| [Routes en lazy loading](best-practices/performance/lazy-loaded-routes#lazily-loaded-components-and-routes) | Diffère le chargement des composants de route jusqu'à la navigation, ce qui réduit la taille du bundle initial | Applications avec plusieurs routes qui ne sont pas toutes nécessaires au chargement initial |
| [Chargement différé avec `@defer`](best-practices/performance/defer) | Découpe les composants en bundles séparés chargés à la demande | Composants non visibles au rendu initial, libraries tierces lourdes, contenu sous la ligne de flottaison |
| [Lazy loading de services avec `injectAsync`](guide/di/lazy-loading-services) | Découpe les services rarement utilisés en chunks séparés et les charge à la demande | Services reposant sur de grosses libraries ou fonctionnalités peu utilisées |
| [Optimisation des images](best-practices/performance/image-optimization) | Donne la priorité aux images LCP, charge les autres en lazy loading, génère des attributs `srcset` responsives | Toute application qui affiche des images |
| [Rendu côté serveur](best-practices/performance/ssr) | Effectue le rendu des pages sur le serveur pour un premier affichage plus rapide et un meilleur référencement (SEO), avec l'[hydratation](guide/hydration) pour restaurer l'interactivité et l'[hydratation incrémentale](guide/incremental-hydration) pour différer l'hydratation de certaines sections jusqu'à ce qu'elles soient nécessaires | Applications riches en contenu, pages qui doivent être indexées par les moteurs de recherche |

## Performances à l'exécution {#runtime-performance}

Les performances à l'exécution déterminent la réactivité de votre application une fois chargée. Le système de détection de changements d'Angular maintient le DOM synchronisé avec vos données, et optimiser comment et quand il s'exécute est le principal levier pour améliorer les performances à l'exécution.

| Technique | Ce qu'elle fait | Quand l'utiliser |
| :-------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------ |
| [Détection de changements zoneless](guide/zoneless) | Supprime le surcoût de ZoneJS et ne déclenche la détection de changements que lorsque des signals ou des événements indiquent un changement | Nouvelles applications (par défaut à partir d'Angular v21), ou applications existantes prêtes à migrer |
| [Calculs lents](best-practices/slow-computations) | Identifie et optimise les expressions de template et les lifecycle hooks coûteux | Le profilage révèle des composants précis qui ralentissent les cycles de détection de changements |
| [Ignorer des sous-arbres de composants](best-practices/skipping-subtrees) | Utilise la détection de changements `OnPush` pour ignorer les arbres de composants inchangés | Applications qui ont besoin d'un contrôle plus fin sur la détection de changements |
| [Pollution de la zone](best-practices/zone-pollution) | Empêche la détection de changements inutile causée par des libraries tierces ou des timers | Applications basées sur Zone.js où le profilage révèle des cycles de détection de changements excessifs |

## Mesurer les performances {#measuring-performance}

Identifier ce qu'il faut optimiser est tout aussi important que savoir comment l'optimiser. Angular s'intègre aux outils de développement du navigateur pour vous aider à trouver les goulots d'étranglement.

| Outil | Ce qu'il fait |
| :------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Profilage avec Chrome DevTools](best-practices/profiling-with-chrome-devtools) | Enregistre des données de performance propres à Angular en parallèle du profilage du navigateur, avec des flame charts colorés qui montrent le rendu des composants, les cycles de détection de changements et les lifecycle hooks |
| [Angular DevTools](tools/devtools) | Une extension de navigateur qui fournit un inspecteur de l'arbre des composants et un profiler pour visualiser les cycles de détection de changements |

## Que faut-il optimiser en premier ? {#what-to-optimize-first}

Si vous ne savez pas par où commencer, profilez d'abord votre application avec la [piste Angular de Chrome DevTools](best-practices/profiling-with-chrome-devtools) pour identifier les goulots d'étranglement précis.

Comme point de départ général :

- **Chargement initial lent** — Utilisez [`@defer`](best-practices/performance/defer) pour sortir les gros composants du bundle principal, [`NgOptimizedImage`](best-practices/performance/image-optimization) pour donner la priorité aux images au-dessus de la ligne de flottaison, et le [rendu côté serveur](best-practices/performance/ssr) pour livrer le contenu plus rapidement.
- **Interactions lentes après le chargement** — Vérifiez si la [détection de changements zoneless](guide/zoneless) est activée, recherchez les [calculs lents](best-practices/slow-computations) dans les templates ou les lifecycle hooks, et envisagez [`OnPush`](best-practices/skipping-subtrees) pour réduire la détection de changements inutile.
