<!-- source: d8e7ddec40a7e84a2851887b1f8c59c379b021c3 -->
<docs-decorative-header title="Feuille de route d'Angular" imgSrc="adev/src/assets/images/roadmap.svg"> <!-- markdownlint-disable-line -->
Découvrez comment l'équipe Angular fait avancer le web.
</docs-decorative-header>

En tant que projet open source, les commits quotidiens, les PR et la dynamique d'Angular sont entièrement traçables sur GitHub. Pour plus de transparence sur la façon dont ce travail quotidien s'inscrit dans l'avenir du framework, notre feuille de route rassemble la vision actuelle et future de l'équipe.

Les projets suivants ne sont pas associés à une version particulière d'Angular. Nous les publierons une fois terminés, et ils feront partie d'une version précise selon notre calendrier de sorties, conformément au versionnage sémantique. Par exemple, nous publions les fonctionnalités dans la version mineure suivant leur achèvement, ou dans la version majeure suivante si elles comportent des changements incompatibles.

Actuellement, Angular poursuit trois objectifs pour le framework :

1. Améliorer l'[expérience de l'IA pour les développeurs](/ai)
1. Améliorer l'[expérience de développement Angular](#improving-the-angular-developer-experience)
1. Améliorer les performances du framework

Poursuivez votre lecture pour découvrir comment nous prévoyons d'atteindre ces objectifs grâce à des projets concrets.

## Explorer Angular moderne {#explore-modern-angular}

Commencez à développer avec les dernières fonctionnalités d'Angular issues de notre feuille de route. Cette liste présente le statut actuel des nouvelles fonctionnalités de notre feuille de route :

### Disponible pour expérimentation {#available-to-experiment-with}

- [Web MCP](/ai/webmcp)

### Prêt pour la production {#production-ready}

- [Signal Forms](/guide/forms/signals/overview)
- [API Resource](/guide/signals/resource)
- [httpResource](/api/common/http/httpResource)
- [Détection de changements zoneless](/guide/zoneless)
- [API Linked Signal](/guide/signals/linked-signal)
- [Hydratation incrémentale](/guide/incremental-hydration)
- [API Effect](/api/core/effect)
- [Event replay avec le SSR](/api/platform-browser/withEventReplay)
- [Mode de rendu au niveau des routes](/guide/ssr)

## Améliorer l'expérience de l'IA pour les développeurs Angular {#improving-the-ai-experience-for-angular-developers}

### Apporter le meilleur de l'IA à Angular {#bringing-the-best-of-ai-to-angular}

<docs-card-container>
  <docs-card title="Angular propulsé par l'IA">
  L'IA continue de transformer le paysage du développement. Elle a changé notre façon de développer des applications et les types d'expériences utilisateur possibles. Nous prévoyons d'accompagner au mieux la communauté des développeurs dans le code assisté par l'IA et l'intégration de l'IA dans leurs applications.
  </docs-card>
  <docs-card title="Développement avec l'IA">
  L'équipe continuera à développer des intégrations utiles avec des outils comme Google AI Studio, Gemini CLI et d'autres outils agentiques, comme les IDE agentiques tels qu'Antigravity. Nous prévoyons de lancer des solutions qui suivent une industrie en évolution rapide. Parmi les exemples figurent les agent skills, de nouvelles fonctionnalités MCP et des SDK d'IA.
  </docs-card>
  <docs-card title="Génération de code">
  [D'après nos recherches](https://blog.angular.dev/beyond-the-horizon-how-angular-is-embracing-ai-for-next-gen-apps-7a7ed706e1a3), la génération de code pour Angular est déjà de grande qualité avec les LLM modernes. Nous continuerons d'investir dans l'amélioration de la génération de code pour Angular. Concrètement, nous évaluerons régulièrement la qualité de la génération de code avec les modèles actuels et travaillerons à l'améliorer grâce aux instructions système, à la documentation et à des changements ciblés dans le framework.
  </docs-card>
  <docs-card title="Expériences propulsées par l'IA">
  Une nouvelle frontière s'ouvre aux développeurs Angular avec de nouveaux concepts comme la génération dynamique d'interfaces. Nous avons commencé par développer la prise en charge d'A2UI dans Angular et recherchons activement d'autres occasions de prendre en charge les expériences applicatives modernes.
  </docs-card>
</docs-card-container>

## Améliorer l'expérience de développement Angular {#improving-the-angular-developer-experience}

### Vélocité des développeurs {#developer-velocity}

<docs-card-container>
  <docs-card title="Compilateur">
    Microsoft a passé l'année dernière à porter le compilateur TypeScript en Go, avec la promesse d'une accélération de 5 à 10 fois pour les compilations TypeScript typiques. Angular possède peut-être l'une des intégrations les plus profondes avec le compilateur TypeScript, ce qui nécessitera des changements d'architecture plus importants pour prendre en charge les nouveaux workflows basés sur tsgo, à la fois pour le compilateur et pour le Language Service.

Nous sommes en train de prototyper et d'explorer à quoi ressemblerait cette prise en charge, et nous livrerons un compilateur Angular compatible avec tsgo qui apportera à l'écosystème Angular les gains de performances du portage natif de Microsoft.
</docs-card>

  <docs-card title="Compatibilité accrue avec l'écosystème">
    Les développeurs mélangent du code généré par l'IA avec du code écrit à la main, et veulent utiliser des libraries populaires et intégrer rapidement de nouvelles expériences. Angular veut bien s'intégrer à cet écosystème : les développeurs doivent pouvoir utiliser les outils qu'ils aiment et combiner les frameworks selon leurs besoins.

Dans le cadre de ce projet, nous explorerons les besoins en matière d'interopérabilité entre frameworks ainsi que notre outillage de build afin d'améliorer notre compatibilité.

  </docs-card>

  <docs-card title="Composants">
  Dans Angular v21, nous avons lancé Angular Aria en developer preview, avec huit patterns pour des composants accessibles et headless. Dans Angular v22, nous avons promu Angular Aria en version stable, et nous prévoyons d'introduire de nouveaux patterns si nécessaire. Nous voulons offrir aux développeurs une base solide pour développer leurs propres composants avec Angular Aria : nous fournissons les interactions et vous apportez le style qui correspond à vos design systems. Les développeurs auront le choix de développer des composants personnalisés avec Angular Aria, d'utiliser les patterns d'interaction du CDK ou d'utiliser les composants Material prêts à l'emploi et déjà stylés.

En matière d'accessibilité, nous évaluons en continu les composants et les patterns au regard des normes d'accessibilité comme les WCAG, et nous travaillons à corriger les problèmes qui en ressortent.
</docs-card>
</docs-card-container>

### Améliorer l'outillage {#improve-tooling}

<docs-card-container>
  <docs-card title="Moderniser l'outillage de tests unitaires avec ng test">
  Après la sortie stable de Vitest dans Angular v21, c'est désormais notre test runner principal. Nous nous concentrons maintenant sur la stabilisation de notre outil expérimental de migration de Karma vers Vitest, ainsi que sur l'étude de nouvelles fonctionnalités pour affiner et améliorer encore le workflow de test des développeurs.
</docs-card>
</docs-card-container>

## Projets terminés {#completed-projects}

<docs-card-container>

  <docs-card title="Signal Forms" href="/guide/forms/signals/overview" link="Terminé en 2026">
  Les Signal Forms sont désormais stables. Cette nouvelle approche permet aux développeurs de gérer l'état des formulaires avec des signals, pour une expérience de création de formulaires ergonomique. Nous avons promu les Signal Forms en version stable et amélioré l'interopérabilité avec les reactive forms, ce qui permet aux équipes de migrer progressivement de grands formulaires à leur propre rythme.
  </docs-card>

  <docs-card title="Réactivité" href="/guide/signals" link="Terminé en 2026">
  Nous avons introduit de nouvelles API de signals, `resource()` et `httpResource()`, pour une gestion flexible des données asynchrones. Nous avons promu ces API en version stable pour les développeurs.
  </docs-card>

  <docs-card title="Détection de changements" href="/api/core/ChangeDetectionStrategy" link="Terminé en 2026">
  Le mode zoneless étant stable et activé par défaut, nous avons défini `OnPush` comme stratégie de détection de changements par défaut, conformément aux bonnes pratiques actuelles. Nous avons également renommé `ChangeDetectionStrategy.Default` en `ChangeDetectionStrategy.Eager`.

[Consultez la discussion de la RFC pour plus de détails](https://github.com/angular/angular/discussions/66779).
</docs-card>

  <docs-card title="Débogage des signals dans Angular DevTools" link="Terminé en 2026">
  Nous avons ajouté un meilleur outillage pour déboguer les signals avec Angular DevTools. Ce changement comprend une nouvelle interface pour inspecter et déboguer les signals.
  </docs-card>

  <docs-card title="Améliorer le HMR (Hot Module Reload)" href="https://github.com/angular/angular/issues/39367#issuecomment-1439537306" link="Terminé en 2025">
  Nous travaillons à accélérer le cycle modification/rafraîchissement en activant le remplacement de modules à chaud (hot module replacement). Dans Angular v19, nous avons livré une première prise en charge du HMR pour le CSS et les templates, et dans la v20 nous avons promu le HMR des templates en version stable. Nous continuerons à recueillir des retours pour nous assurer de répondre aux besoins des développeurs avant de marquer ce projet comme terminé.
</docs-card>

<docs-card title="Angular zoneless"  link="Terminé au T4 2025">
Dans la v18, nous avons livré une prise en charge expérimentale du mode zoneless dans Angular. Elle permet aux développeurs d'utiliser le framework sans inclure zone.js dans leur bundle, ce qui améliore les performances, l'expérience de débogage et l'interopérabilité. Dans le cadre de cette première version, nous avons aussi introduit la prise en charge du mode zoneless dans l'Angular CDK et Angular Material.

Dans la v19, nous avons introduit la prise en charge du mode zoneless pour le rendu côté serveur, traité certains cas limites et créé un schematic pour générer des projets zoneless. Nous avons migré <a href="https://fonts.google.com/" target="_blank">Google Fonts</a> vers le mode zoneless, ce qui a amélioré les performances et l'expérience de développement, et nous a permis d'identifier les lacunes à combler avant de passer cette fonctionnalité en developer preview.

Depuis Angular v20.2, Angular zoneless est stable et comprend des améliorations de la gestion des erreurs et du rendu côté serveur.
</docs-card>

<docs-card title="Configuration des routes serveur" link="Terminé au T2 2025"  >
Nous travaillons à rendre la configuration des routes côté serveur plus ergonomique. Nous voulons qu'il soit trivial de déclarer quelles routes doivent être rendues côté serveur, prérendues ou rendues côté client.

Dans Angular v19, nous avons livré en developer preview le mode de rendu au niveau des routes, qui vous permet de configurer finement quelles routes Angular doit prérendre, rendre côté serveur ou rendre côté client. Dans Angular v20, nous l'avons promu en version stable.
</docs-card>
<docs-card title="Activer l'hydratation incrémentale" link="Terminé au T2 2025">
Dans la v17, nous avons fait sortir l'hydratation de la developer preview et nous observons régulièrement des améliorations de 40 à 50 % du LCP. Depuis, nous avons commencé à prototyper l'hydratation incrémentale et en avons présenté une démo sur scène à ng-conf.

Dans la v19, nous avons livré l'hydratation incrémentale en developer preview, reposant sur les blocs `@defer`. Dans Angular v20, nous l'avons promue en version stable !
</docs-card>
<docs-card title="Livrer les signals Angular" link="Terminé au T2 2025" href="https://github.com/angular/angular/discussions/49685">
Ce projet repense le modèle de réactivité d'Angular en introduisant les signals comme primitive de réactivité. La planification initiale a donné lieu à des centaines de discussions, des échanges avec des développeurs, des sessions de retours, des études d'expérience utilisateur et une série de RFC, qui ont reçu plus de 1 000 commentaires.

Dans Angular v20, nous avons promu en version stable toutes les primitives de réactivité fondamentales, notamment signal, effect, linkedSignal, les queries basées sur les signals et les inputs.
</docs-card>
<docs-card title="Prendre en charge le glisser-déposer en deux dimensions" link="Terminé au T2 2024" href="https://github.com/angular/components/issues/13372">
Dans le cadre de ce projet, nous avons implémenté la prise en charge de l'orientation mixte pour le glisser-déposer de l'Angular CDK. C'est l'une des fonctionnalités les plus demandées du dépôt.
</docs-card>
<docs-card title="Event replay avec le SSR et le prérendu" link="Terminé au T4 2024" href="api/platform-browser/withEventReplay">
Dans la v18, nous avons introduit une fonctionnalité d'event replay lors de l'utilisation du rendu côté serveur ou du prérendu. Pour cette fonctionnalité, nous nous appuyons sur la primitive event dispatch (anciennement appelée jsaction) qui s'exécute sur Google.com.

Dans Angular v19, nous avons promu l'event replay en version stable et l'avons activé par défaut pour tous les nouveaux projets.
</docs-card>
<docs-card title="Intégrer l'Angular Language Service aux schematics" link="Terminé au T4 2024">
Pour faciliter l'utilisation des API Angular modernes, nous avons activé l'intégration entre l'Angular Language Service et les schematics, ce qui vous permet de refactoriser votre application en un seul clic.
</docs-card>
<docs-card title="Simplifier les imports standalone avec le Language Service" link="Terminé au T4 2024">
Dans le cadre de cette initiative, le Language Service importe automatiquement les composants et les pipes dans les applications standalone et basées sur NgModule. Nous avons également ajouté un diagnostic de template qui signale les imports inutilisés dans les composants standalone, ce qui devrait contribuer à réduire la taille des bundles des applications.
</docs-card>
<docs-card title="Variables locales de template" link="Terminé au T3 2024">
Nous avons publié la prise en charge des variables locales de template dans Angular ; consultez [`@let`](/api/core/@let) pour plus d'informations.
</docs-card>
<docs-card title="Étendre les possibilités de personnalisation d'Angular Material" link="Terminé au T2 2024" href="https://material.angular.dev/guide/theming">
Pour mieux personnaliser nos composants Angular Material et activer les capacités de Material 3, nous collaborerons avec l'équipe Material Design de Google pour définir des API de thèmes basées sur des tokens.

Dans la v17.2, nous avons proposé une prise en charge expérimentale d'Angular Material 3, et dans la v18 nous l'avons promue en version stable.
</docs-card>
<docs-card title="Introduire le chargement différé" link="Terminé au T2 2024" href="guide/templates/defer">
Dans la v17, nous avons livré les vues différables en developer preview, qui offrent une API ergonomique pour le chargement différé du code. Dans la v18, nous avons activé les vues différables pour les développeurs de libraries et promu l'API en version stable.
</docs-card>
<docs-card title="Prise en charge des iframes dans Angular DevTools" link="Terminé au T2 2024">
Nous avons rendu possible le débogage et le profilage des applications Angular intégrées dans une iframe de la page.
</docs-card>
<docs-card title="Automatiser la migration des projets existants à rendu hybride vers esbuild et Vite" link="Terminé au T2 2024" href="tools/cli/build-system-migration">
Dans la v17, nous avons livré un builder d'application basé sur Vite et esbuild, et l'avons activé par défaut pour les nouveaux projets. Il améliore jusqu'à 87 % le temps de build des projets utilisant le rendu hybride. Dans la v18, nous avons livré des schematics et un guide qui migrent les projets existants utilisant le rendu hybride vers le nouveau pipeline de build.
</docs-card>
<docs-card title="Faire d'Angular.dev la maison officielle des développeurs Angular" link="Terminé au T2 2024" href="https://goo.gle/angular-dot-dev">
Angular.dev est le nouveau site, domaine et point d'ancrage du développement Angular. Le nouveau site contient une documentation, des tutoriels et des conseils à jour qui aideront les développeurs à construire avec les dernières fonctionnalités d'Angular.
</docs-card>
<docs-card title="Introduire le control flow intégré" link="Terminé au T2 2024" href="guide/templates/control-flow">
Dans la v17, nous avons livré une version developer preview d'un nouveau control flow. Il apporte d'importantes améliorations de performances et une meilleure ergonomie pour l'écriture des templates. Nous avons aussi fourni une migration des `*ngIf`, `*ngFor` et `*ngSwitch` existants, que vous pouvez exécuter pour faire passer votre projet à la nouvelle implémentation. Depuis la v18, le control flow intégré est stable.
</docs-card>
<docs-card title="Moderniser le tutoriel de prise en main" link="Terminé au T4 2023">
Au cours des deux derniers trimestres, nous avons développé un nouveau tutoriel [vidéo](https://www.youtube.com/watch?v=xAT0lHYhHMY&list=PL1w1q3fL4pmj9k1FrJ3Pe91EPub2_h4jF) et [écrit](/tutorials/learn-angular) basé sur les composants standalone.
</docs-card>
<docs-card title="Étudier les bundlers modernes" link="Terminé au T4 2023" href="guide/hydration">
Dans Angular v16, nous avons publié une developer preview d'un builder basé sur esbuild prenant en charge `ng build` et `ng serve`. Le serveur de développement `ng serve` utilise Vite et une compilation multifichier par esbuild et le compilateur Angular. Dans la v17, nous avons fait sortir l'outillage de build de la developer preview et l'avons activé par défaut pour les nouveaux projets.
</docs-card>
<docs-card title="Introduire des API de débogage de l'injection de dépendances" link="Terminé au T4 2023" href="tools/devtools">
Pour améliorer les outils de débogage d'Angular et d'Angular DevTools, nous travaillerons sur des API donnant accès au runtime de l'injection de dépendances. Dans le cadre du projet, nous exposerons des méthodes de débogage qui permettent d'explorer la hiérarchie des injectors et les dépendances à travers leurs providers associés. Depuis la v17, nous avons livré une fonctionnalité qui nous permet de nous brancher sur le cycle de vie de l'injection de dépendances. Nous avons aussi lancé une visualisation de l'arbre des injectors et l'inspection des providers déclarés dans chaque injector individuel,
</docs-card>
<docs-card title="Améliorer la documentation et les schematics pour les composants standalone" link="Terminé au T4 2023" href="essentials/components">
Nous avons publié une developer preview de la collection de schematics `ng new --standalone`, qui vous permet de créer des applications sans NgModules. Dans la v17, nous avons fait passer le format de création des nouvelles applications aux API standalone et modifié la documentation pour refléter cette recommandation. Nous avons également livré des schematics qui permettent de migrer les applications existantes vers des composants, directives et pipes standalone. Même si les NgModules resteront présents dans un avenir prévisible, nous vous recommandons d'explorer les avantages des nouvelles API pour améliorer l'expérience de développement et profiter des nouvelles fonctionnalités que nous construisons pour elles.
</docs-card>
<docs-card title="Explorer les améliorations de l'hydratation et du rendu côté serveur" link="Terminé au T4 2023">
Dans la v16, nous avons publié une developer preview de l'hydratation complète non destructive ; consultez le [guide de l'hydratation](guide/hydration) et l'[article de blog](https://blog.angular.dev/whats-next-for-server-side-rendering-in-angular-2a6f27662b67) pour plus d'informations. Nous constatons déjà des améliorations significatives des Core Web Vitals, notamment du [LCP](https://web.dev/lcp) et du [CLS](https://web.dev/cls). Lors de tests en laboratoire, nous avons constamment observé un LCP amélioré de 45 % sur une application réelle.

Dans la v17, nous avons fait sortir l'hydratation de la developer preview et apporté une série d'améliorations au rendu côté serveur, notamment : la découverte des routes à l'exécution pour le SSG, des temps de build jusqu'à 87 % plus rapides pour les applications à rendu hybride, et une invite qui active le rendu hybride pour les nouveaux projets.
</docs-card>
<docs-card title="Hydratation complète et non destructive des applications" link="Terminé au T1 2023" href="guide/hydration">
Dans la v16, nous avons publié une developer preview de l'hydratation complète non destructive, qui permet à Angular de réutiliser les nœuds DOM existants d'une page rendue côté serveur au lieu de recréer une application à partir de zéro. Consultez le guide de l'hydratation pour plus d'informations.
</docs-card>
<docs-card title="Améliorations de la directive d'image" link="Terminé au T1 2023" href="guide/image-optimization">
Nous avons publié la directive d'image d'Angular en version stable dans la v15. Nous avons introduit un nouveau mode fill qui permet aux images de s'adapter à leur conteneur parent au lieu d'avoir des dimensions explicites. Au cours des deux derniers mois, l'équipe Chrome Aurora a rétroporté la directive vers la v12 et les versions ultérieures.
</docs-card>
<docs-card title="Refonte de la documentation" link="Terminé au T1 2023" href="https://angular.io">
Faire en sorte que toute la documentation existante s'inscrive dans un ensemble cohérent de types de contenus. Transformer la documentation trop orientée tutoriel en sujets indépendants. Nous voulons que le contenu en dehors des tutoriels principaux soit autonome, sans être étroitement lié à une série de guides. Au T2 2022, nous avons refondu le contenu sur les templates et l'injection de dépendances. Au T1 2023, nous avons amélioré les guides HTTP et, avec cela, nous mettons en pause le projet de refonte de la documentation.
</docs-card>
<docs-card title="Améliorer les performances des images" link="Terminé au T4 2022" href="guide/image-optimization">
Les équipes Aurora et Angular travaillent à l'implémentation d'une directive d'image visant à améliorer les Core Web Vitals. Nous avons livré une version stable de la directive d'image dans la v15.
</docs-card>
<docs-card title="CSS moderne" link="Terminé au T4 2022" href="https://blog.angular.dev/modern-css-in-angular-layouts-4a259dca9127">
L'écosystème web évolue constamment et nous voulons refléter les standards modernes les plus récents dans Angular. Dans ce projet, nous voulons fournir des recommandations sur l'utilisation des fonctionnalités CSS modernes dans Angular, afin que les développeurs suivent les bonnes pratiques de mise en page, de style, etc. Nous avons partagé des recommandations officielles pour la mise en page et, dans le cadre de cette initiative, avons cessé de publier flex layout.
</docs-card>
<docs-card title="Permettre d'ajouter des directives aux host elements" link="Terminé au T4 2022" href="guide/directives/directive-composition-api">
Une demande de fonctionnalité de longue date consistait à pouvoir ajouter des directives aux host elements. Cette fonctionnalité permet aux développeurs d'enrichir leurs propres composants avec des comportements supplémentaires sans utiliser l'héritage. Dans la v15, nous avons livré notre API de composition de directives, qui permet d'enrichir les host elements avec des directives.
</docs-card>
<docs-card title="De meilleures stack traces" link="Terminé au T4 2022" href="https://developer.chrome.com/blog/devtools-better-angular-debugging/">
Les équipes Angular et Chrome DevTools travaillent ensemble pour rendre les stack traces des messages d'erreur plus lisibles. Dans la v15, nous avons publié des stack traces plus pertinentes et liées. En tant qu'initiative de moindre priorité, nous étudierons comment rendre les stack traces plus conviviales en fournissant des noms de frames d'appel plus précis pour les templates.
</docs-card>
<docs-card title="Composants Angular Material améliorés grâce à l'intégration de MDC Web" link="Terminé au T4 2022" href="https://material.angular.dev/guide/mdc-migration">
MDC Web est une library créée par l'équipe Material Design de Google qui fournit des primitives réutilisables pour construire des composants Material Design. L'équipe Angular intègre ces primitives dans Angular Material. L'utilisation de MDC Web rapproche Angular Material de la spécification Material Design, étend l'accessibilité, améliore la qualité des composants et accélère le rythme de notre équipe.
</docs-card>
<docs-card title="Implémenter des API pour des NgModules facultatifs" link="Terminé au T4 2022" href="https://blog.angular.dev/angular-v15-is-now-available-df7be7f2f4c8">
Pour rendre Angular plus simple, nous travaillons à introduire des API qui permettent aux développeurs d'initialiser des applications, d'instancier des composants et d'utiliser le router sans NgModules. Angular v14 introduit en developer preview les API pour les composants, directives et pipes standalone. Au cours des prochains trimestres, nous recueillerons les retours des développeurs et finaliserons le projet en rendant les API stables. Ensuite, nous travaillerons à améliorer des cas d'usage comme TestBed, Angular Elements, etc.
</docs-card>
<docs-card title="Permettre le binding sur des champs protected dans les templates" link="Terminé au T2 2022" href="guide/templates/binding">
Pour améliorer l'encapsulation des composants Angular, nous avons permis le binding sur les membres protected de l'instance du composant. Ainsi, vous n'aurez plus besoin d'exposer un champ ou une méthode en public pour l'utiliser dans vos templates.
</docs-card>
<docs-card title="Publier des guides sur les concepts avancés" link="Terminé au T2 2022" href="https://angular.io/guide/change-detection">
Rédiger et publier un guide approfondi sur la détection de changements. Produire du contenu sur le profilage des performances des applications Angular. Expliquer comment la détection de changements interagit avec Zone.js, quand elle est déclenchée, comment profiler sa durée, ainsi que les pratiques courantes d'optimisation des performances.
</docs-card>
<docs-card title="Déployer le typage strict pour @angular/forms" link="Terminé au T2 2022" href="guide/forms/typed-forms">
Au T4 2021, nous avons conçu une solution pour introduire le typage strict des formulaires et, au T1 2022, nous avons clôturé la demande de commentaires (RFC) correspondante. Nous mettons actuellement en œuvre une stratégie de déploiement avec une étape de migration automatisée qui activera les améliorations pour les projets existants. Nous testons d'abord la solution sur plus de 2 500 projets chez Google pour garantir un chemin de migration fluide à la communauté externe.
</docs-card>
<docs-card title="Supprimer l'ancien View Engine" link="Terminé au T1 2022" href="https://blog.angular.dev/angular-v15-is-now-available-df7be7f2f4c8">
Une fois la transition de tout notre outillage interne vers Ivy terminée, nous supprimerons l'ancien View Engine afin de réduire la charge conceptuelle d'Angular, la taille du package, le coût de maintenance et la complexité de la base de code.
</docs-card>
<docs-card title="Un modèle mental d'Angular simplifié avec des NgModules facultatifs" link="Terminé au T1 2022" href="https://blog.angular.dev/angular-v15-is-now-available-df7be7f2f4c8">
Pour simplifier le modèle mental d'Angular et son apprentissage, nous travaillerons à rendre les NgModules facultatifs. Ce travail permet aux développeurs de développer des composants standalone et d'implémenter une API alternative pour déclarer la portée de compilation du composant. Nous avons lancé ce projet par des discussions de conception de haut niveau, que nous avons consignées dans une RFC.
</docs-card>
<docs-card title="Concevoir un typage strict pour @angular/forms" link="Terminé au T1 2022" href="guide/forms/typed-forms">
Nous chercherons un moyen d'implémenter une vérification de types plus stricte pour les reactive forms avec un minimum d'impacts non rétrocompatibles. Ainsi, les développeurs pourront détecter davantage de problèmes pendant le développement, bénéficier d'une meilleure prise en charge par les éditeurs de texte et les IDE, et d'une vérification de types améliorée pour les reactive forms.
</docs-card>
<docs-card title="Améliorer l'intégration d'Angular DevTools au framework" link="Terminé au T1 2022" href="tools/devtools">
Pour améliorer l'intégration d'Angular DevTools au framework, nous travaillons à déplacer la base de code dans le monorepo angular/angular. Cela comprend la migration d'Angular DevTools vers Bazel et son intégration aux processus et au pipeline de CI existants.
</docs-card>
<docs-card title="Lancer des diagnostics avancés du compilateur" link="Terminé au T1 2022" href="extended-diagnostics">
Étendre les diagnostics du compilateur Angular au-delà de la vérification de types. Introduire d'autres vérifications d'exactitude et de conformité pour mieux garantir l'exactitude du code et le respect des bonnes pratiques.
</docs-card>
<docs-card title="Mettre à jour notre stratégie de tests e2e" link="Terminé au T3 2021" href="guide/testing">
Pour proposer une stratégie de tests e2e pérenne, nous voulons évaluer l'état de Protractor, les innovations de la communauté et les bonnes pratiques e2e, et explorer de nouvelles pistes. Comme premières étapes, nous avons partagé une RFC et travaillé avec des partenaires pour assurer une intégration fluide entre l'Angular CLI et les outils de pointe pour les tests e2e. Ensuite, nous devons finaliser les recommandations et compiler une liste de ressources pour la transition.
</docs-card>
<docs-card title="Les libraries Angular utilisent Ivy" link="Terminé au T3 2021" href="tools/libraries">
Début 2020, nous avons partagé une RFC sur la distribution des libraries Ivy. Après des retours précieux de la communauté, nous avons élaboré une conception du projet. Nous investissons maintenant dans le développement de la distribution des libraries Ivy, y compris une mise à jour du format de package des libraries pour utiliser la compilation Ivy, le déblocage de la dépréciation du format de library View Engine, et ngcc.
</docs-card>
<docs-card title="Améliorer la durée des tests et le débogage grâce au nettoyage automatique de l'environnement de test" link="Terminé au T3 2021" href="guide/testing">
Pour améliorer la durée des tests et mieux isoler les tests entre eux, nous voulons modifier TestBed afin qu'il nettoie et démonte automatiquement l'environnement de test après chaque exécution de test.
</docs-card>
<docs-card title="Déprécier et supprimer la prise en charge d'IE11" link="Terminé au T3 2021" href="https://github.com/angular/angular/issues/41840">
Internet Explorer 11 (IE11) empêchait Angular de tirer parti de certaines fonctionnalités modernes de la plateforme web. Dans le cadre de ce projet, nous allons déprécier puis supprimer la prise en charge d'IE11 pour ouvrir la voie aux fonctionnalités modernes qu'offrent les navigateurs evergreen. Nous avons mené une RFC pour recueillir les retours de la communauté et décider des prochaines étapes.
</docs-card>
<docs-card title="Utiliser ES2017+ comme langage de sortie par défaut" link="Terminé au T3 2021" href="https://www.typescriptlang.org/docs/handbook/tsconfig-json.html">
La prise en charge des navigateurs modernes nous permet de tirer parti de la nouvelle syntaxe JavaScript, plus compacte, plus expressive et plus performante. Dans le cadre de ce projet, nous étudierons ce qui bloque cet effort et prendrons les mesures nécessaires pour le rendre possible.
</docs-card>
<docs-card title="Débogage et profilage des performances accélérés avec Angular DevTools" link="Terminé au T2 2021" href="tools/devtools">
Nous travaillons sur un outillage de développement pour Angular qui fournit des utilitaires de débogage et de profilage des performances. Ce projet vise à aider les développeurs à comprendre la structure des composants et la détection de changements dans une application Angular.
</docs-card>
<docs-card title="Simplifier les sorties grâce à une gestion des versions et des branches Angular unifiée" link="Terminé au T2 2021" href="reference/releases">
Nous voulons unifier l'outillage de gestion des sorties entre les différents dépôts GitHub d'Angular (angular/angular, angular/angular-cli et angular/components). Cet effort nous permet de réutiliser l'infrastructure, d'unifier et de simplifier les processus, et d'améliorer la fiabilité de notre processus de sortie.
</docs-card>
<docs-card title="Plus de cohérence pour les développeurs grâce à la standardisation des messages de commit" link="Terminé au T2 2021" href="https://github.com/angular/angular">
Nous voulons unifier les exigences et la conformité des messages de commit dans les dépôts Angular (angular/angular, angular/components et angular/angular-cli) afin d'apporter de la cohérence à notre processus de développement et de réutiliser l'outillage d'infrastructure.
</docs-card>
<docs-card title="Faire passer l'Angular Language Service à Ivy" link="Terminé au T2 2021" href="tools/language-service">
L'objectif de ce projet est d'améliorer l'expérience et de supprimer une dépendance héritée en faisant passer le Language Service à Ivy. Aujourd'hui, le Language Service utilise encore le compilateur et la vérification de types de View Engine, même pour les applications Ivy. Nous voulons utiliser le parser de templates d'Ivy et sa vérification de types améliorée pour l'Angular Language Service, afin de correspondre au comportement de l'application. Cette migration est aussi une étape vers le déblocage de la suppression de View Engine, qui simplifiera Angular, réduira la taille du package npm et améliorera la maintenabilité du framework.
</docs-card>
<docs-card title="Une sécurité renforcée grâce aux Trusted Types natifs dans Angular" link="Terminé au T2 2021" href="best-practices/security">
En collaboration avec l'équipe sécurité de Google, nous ajoutons la prise en charge de la nouvelle API Trusted Types. Cette API de la plateforme web aide les développeurs à construire des applications web plus sûres.
</docs-card>
<docs-card title="Vitesse de build et taille des bundles optimisées avec webpack 5 dans l'Angular CLI" link="Terminé au T2 2021" href="tools/cli/build">
Dans le cadre de la v11, nous avons introduit une préversion optionnelle de webpack 5 dans l'Angular CLI. Pour garantir la stabilité, nous continuerons d'itérer sur l'implémentation afin d'améliorer la vitesse de build et la taille des bundles.
</docs-card>
<docs-card title="Des applications plus rapides grâce à l'intégration des styles critiques dans les applications Universal" link="Terminé au T1 2021" href="guide/ssr">
Le chargement de feuilles de style externes est une opération bloquante : le navigateur ne peut pas commencer à rendre votre application tant qu'il n'a pas chargé tout le CSS référencé. La présence de ressources bloquant le rendu dans l'en-tête d'une page peut avoir un impact important sur ses performances de chargement, par exemple sur son first contentful paint. Pour rendre les applications plus rapides, nous avons collaboré avec l'équipe Google Chrome pour intégrer en ligne le CSS critique et charger le reste des styles de manière asynchrone.
</docs-card>
<docs-card title="Améliorer le débogage grâce à de meilleurs messages d'erreur Angular" link="Terminé au T1 2021" href="errors">
Les messages d'erreur fournissent souvent peu d'informations exploitables pour aider les développeurs à résoudre les problèmes. Nous avons travaillé à rendre les messages d'erreur plus faciles à explorer en ajoutant des codes associés, en rédigeant des guides et d'autres contenus pour faciliter le débogage.
</docs-card>
<docs-card title="Une meilleure prise en main pour les développeurs grâce à une documentation d'introduction remaniée" link="Terminé au T1 2021" href="tutorials">
Nous redéfinirons les parcours d'apprentissage des utilisateurs et remanierons la documentation d'introduction. Nous expliquerons clairement les avantages d'Angular et comment explorer ses capacités, et nous fournirons des conseils pour que les développeurs maîtrisent le framework le plus rapidement possible.
</docs-card>
<docs-card title="Étendre les bonnes pratiques des harnais de composant" link="Terminé au T1 2021" href="https://material.angular.dev/guide/using-component-harnesses">
L'Angular CDK a introduit le concept de harnais de test de composant dans Angular avec la version 9. Les harnais de test permettent aux auteurs de composants de créer des API prises en charge pour tester les interactions avec les composants. Nous continuons d'améliorer cette infrastructure de harnais et de clarifier les bonnes pratiques concernant leur utilisation. Nous travaillons également à favoriser l'adoption des harnais au sein de Google.
</docs-card>
<docs-card title="Rédiger un guide sur la projection de contenu" link="Terminé au T2 2021" href="https://angular.io/docs">
La projection de contenu est un concept central d'Angular qui n'occupe pas la place qu'il mérite dans la documentation. Dans le cadre de ce projet, nous voulons identifier les cas d'usage et concepts clés de la projection de contenu et les documenter.
</docs-card>
<docs-card title="Migrer vers ESLint" link="Terminé au T4 2020" href="tools/cli">
Avec la dépréciation de TSLint, nous passerons à ESLint. Dans le cadre de ce processus, nous veillerons à assurer la rétrocompatibilité avec notre configuration TSLint recommandée actuelle, nous mettrons en place une stratégie de migration pour les applications Angular existantes et nous introduirons de nouveaux outils dans la chaîne d'outils de l'Angular CLI.
</docs-card>
<docs-card title="Opération Bye Bye Backlog (aussi appelée Opération Byelog)" link="Terminé au T4 2020" href="https://github.com/angular/angular/issues">
Nous consacrons activement jusqu'à 50 % de notre capacité d'ingénierie au tri des issues et des PR, jusqu'à ce que nous ayons une compréhension claire des besoins plus larges de la communauté. Ensuite, nous consacrerons jusqu'à 20 % de notre capacité d'ingénierie au traitement rapide des nouvelles contributions.
</docs-card>
</docs-card-container>
