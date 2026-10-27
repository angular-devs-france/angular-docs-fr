<!-- source: 8fae0a4e51e96dc20ac140cf2542c9eb7e39e8c7 -->
<!-- TODO: need an Angular + AI logo -->

<docs-decorative-header title="Construire avec l'IA" imgSrc="adev/src/assets/images/what_is_angular.svg"> <!-- markdownlint-disable-line -->
Construisez des applications propulsées par l'IA. Développez plus vite avec l'IA.
</docs-decorative-header>

HELPFUL: Vous souhaitez commencer à développer dans votre IDE propulsé par l'IA préféré ? <br>Consultez nos [règles de prompts et bonnes pratiques](/ai/develop-with-ai).

L'IA générative (GenAI) avec les grands modèles de langage (LLM) permet de créer des expériences applicatives sophistiquées et engageantes, notamment du contenu personnalisé, des recommandations intelligentes, la génération et la compréhension de médias, le résumé d'informations et des fonctionnalités dynamiques.

Développer de telles fonctionnalités exigeait auparavant une expertise approfondie du domaine et un effort d'ingénierie important. Cependant, de nouveaux produits et SDK abaissent la barrière à l'entrée. Angular est bien adapté à l'intégration de l'IA dans votre application web grâce à :

- des API de templating robustes qui permettent de créer des interfaces dynamiques et proprement composées à partir de contenu généré
- une architecture solide, basée sur les signals, conçue pour gérer dynamiquement les données et l'état
- une intégration transparente d'Angular avec les SDK et API d'IA

Ce guide montre comment utiliser [Genkit](/ai#build-ai-powered-applications-with-genkit-and-angular), [Firebase AI Logic](/ai#build-ai-powered-applications-with-firebase-ai-logic-and-angular) et l'[API Gemini](/ai#build-ai-powered-applications-with-gemini-api-and-angular) pour intégrer dès aujourd'hui l'IA à vos applications Angular. Ce guide lance votre parcours de développement d'applications web propulsées par l'IA en expliquant comment commencer à intégrer l'IA dans des applications Angular. Il partage aussi des ressources, comme des starter kits, des exemples de code et des recettes pour les workflows courants, que vous pouvez utiliser pour monter rapidement en compétence.

Pour commencer, vous devez avoir une compréhension de base d'Angular. Vous découvrez Angular ? Essayez notre [guide des Fondamentaux](/essentials) ou nos [tutoriels de prise en main](/tutorials).

NOTE: Bien que cette page présente des intégrations et des exemples avec des produits d'IA de Google, des outils comme Genkit sont agnostiques vis-à-vis du modèle et vous permettent de choisir votre propre modèle. Dans de nombreux cas, les exemples et échantillons de code s'appliquent à d'autres solutions tierces.

## Premiers pas {#getting-started}

Construire des applications propulsées par l'IA est un domaine nouveau et en évolution rapide. Il peut être difficile de décider par où commencer et quelles technologies choisir. La section suivante propose trois options :

1. _Genkit_ vous laisse le choix [du modèle pris en charge et de l'interface, avec une API unifiée](https://genkit.dev), pour construire des applications full-stack. Idéal pour les applications qui nécessitent une logique d'IA back-end sophistiquée, comme des recommandations personnalisées.

1. _Firebase AI Logic_ fournit une API côté client sécurisée pour les modèles de Google, afin de construire des applications uniquement côté client ou des applications mobiles. Idéal pour des fonctionnalités d'IA interactives directement dans le navigateur, comme l'analyse de texte en temps réel ou des chatbots simples.

1. L'_API Gemini_ vous permet de construire une application qui utilise directement les méthodes et fonctionnalités exposées par l'API, idéal pour les applications full-stack. Adapté aux applications qui ont besoin d'un contrôle direct sur les modèles d'IA, comme la génération d'images personnalisée ou le traitement approfondi de données.

### Construire des applications propulsées par l'IA avec Genkit et Angular {#build-ai-powered-applications-with-genkit-and-angular}

[Genkit](https://genkit.dev) est une boîte à outils open source conçue pour vous aider à construire des fonctionnalités propulsées par l'IA dans des applications web et mobiles. Elle offre une interface unifiée pour intégrer des modèles d'IA de Google, OpenAI, Anthropic, Ollama et d'autres, afin que vous puissiez explorer et choisir les meilleurs modèles pour vos besoins. Comme il s'agit d'une solution côté serveur, vos applications web ont besoin d'un environnement serveur pris en charge, comme un serveur basé sur Node, pour s'intégrer à Genkit. Construire une application full-stack avec Angular SSR vous fournit par exemple le code côté serveur de départ.

Voici des exemples de construction avec Genkit et Angular :

- [Starter kit Agentic Apps with Genkit and Angular](https://github.com/angular/examples/tree/main/genkit-angular-starter-kit) — Vous découvrez le développement avec l'IA ? Commencez ici avec une application simple qui présente un workflow agentique. Le point de départ idéal pour votre première expérience de développement avec l'IA.

- [Utiliser Genkit dans une application Angular](https://genkit.dev/docs/frameworks/angular/) — Construisez une application simple qui utilise les Genkit Flows, Angular et Gemini 2.5 Flash. Ce guide pas à pas vous accompagne dans la création d'une application Angular full-stack dotée de fonctionnalités d'IA.

- [Application Dynamic Story Generator](https://github.com/angular/examples/tree/main/genkit-angular-story-generator) — Apprenez à construire une application Angular agentique propulsée par Genkit, Gemini et Imagen 3 pour générer dynamiquement une histoire selon les interactions de l'utilisateur, avec de superbes panneaux d'images qui accompagnent les événements qui se déroulent. Commencez ici si vous souhaitez expérimenter un cas d'usage plus avancé.

  Cet exemple est aussi accompagné d'une présentation vidéo détaillée de ses fonctionnalités :
  - [Regarder « Building Agentic Apps with Angular and Genkit live ! »](https://youtube.com/live/mx7yZoIa2n4?feature=share)
  - [Regarder « Building Agentic Apps with Angular and Genkit live ! PT 2 »](https://youtube.com/live/YR6LN5_o3B0?feature=share)

- [Building Agentic apps with Firebase and Google Cloud (exemple Barista)](https://developers.google.com/solutions/learn/agentic-barista) - Apprenez à construire une application agentique de commande de café avec Firebase et Google Cloud. Cet exemple utilise à la fois Firebase AI Logic et Genkit.

- [Créer des interfaces dynamiques pilotées par le serveur](https://github.com/angular/examples/tree/main/dynamic-sdui-app) - Apprenez à construire des applications Angular agentiques dont les vues sont générées à l'exécution selon les saisies de l'utilisateur.

  Cet exemple est aussi accompagné d'une présentation vidéo détaillée de ses fonctionnalités :
  - [Regarder « Exploring the future of web apps »](https://www.youtube.com/live/4qargCqOu70?feature=share)

### Construire des applications propulsées par l'IA avec Firebase AI Logic et Angular {#build-ai-powered-applications-with-firebase-ai-logic-and-angular}

[Firebase AI Logic](https://firebase.google.com/products/vertex-ai-in-firebase) offre un moyen sécurisé d'interagir avec la Vertex AI Gemini API ou l'API Imagen directement depuis vos applications web et mobiles. C'est intéressant pour les développeurs Angular, car les applications peuvent être full-stack ou uniquement côté client. Si vous développez une application uniquement côté client, Firebase AI Logic convient bien pour intégrer l'IA à vos applications web.

Voici un exemple de construction avec Firebase AI Logic et Angular :

- [Starter kit Firebase AI Logic x Angular](https://github.com/angular/examples/tree/main/firebase-ai-logic-angular-example) - Utilisez ce starter kit pour construire une application d'e-commerce avec un agent conversationnel capable d'effectuer des tâches. Commencez ici si vous n'avez pas d'expérience avec Firebase AI Logic et Angular.

  Cet exemple inclut une [présentation vidéo détaillée qui explique les fonctionnalités et montre comment en ajouter de nouvelles](https://youtube.com/live/4vfDz2al_BI).

### Construire des applications propulsées par l'IA avec l'API Gemini et Angular {#build-ai-powered-applications-with-gemini-api-and-angular}

L'[API Gemini](https://ai.google.dev/gemini-api/docs) donne accès à des modèles de pointe de Google qui acceptent en entrée de l'audio, des images, de la vidéo et du texte. Ces modèles sont optimisés pour des cas d'usage spécifiques ; [pour en savoir plus, consultez la documentation de l'API Gemini](https://ai.google.dev/gemini-api/docs/models).

- [Template d'application Angular AI Text Editor](https://github.com/FirebaseExtended/firebase-framework-tools/tree/main/starters/angular/ai-text-editor) - Utilisez ce template pour démarrer avec un éditeur de texte entièrement fonctionnel, doté de fonctionnalités propulsées par l'IA comme l'amélioration, l'enrichissement et la formalisation de texte. C'est un bon point de départ pour vous familiariser avec les appels à l'API Gemini via HTTP.

- [Template d'application AI Chatbot](https://github.com/FirebaseExtended/firebase-framework-tools/tree/main/starters/angular/ai-chatbot) - Ce template démarre avec une interface de chatbot qui communique avec l'API Gemini via HTTP.

## Bonnes pratiques {#best-practices}

### Se connecter aux fournisseurs de modèles et sécuriser vos identifiants d'API {#connecting-to-model-providers-and-keeping-your-api-credentials-secure}

Lorsque vous vous connectez à des fournisseurs de modèles, il est important de protéger vos secrets d'API. _Ne placez jamais votre clé d'API dans un fichier envoyé au client, comme `environments.ts`_.

L'architecture de votre application détermine les API et outils d'IA à choisir. Plus précisément, choisissez selon que votre application est côté client ou côté serveur. Des outils comme Firebase AI Logic fournissent une connexion sécurisée aux API des modèles pour le code côté client. Si vous souhaitez utiliser une autre API que Firebase AI Logic ou préférez un autre fournisseur de modèles, envisagez de créer un serveur proxy, voire d'utiliser [Cloud Functions for Firebase](https://firebase.google.com/docs/functions) comme proxy, afin de ne pas exposer vos clés d'API.

Pour un exemple de connexion depuis une application côté client, consultez le code : [dépôt d'exemple Firebase AI Logic Angular](https://github.com/angular/examples/tree/main/firebase-ai-logic-angular-example).

Pour les connexions côté serveur aux API de modèles qui exigent des clés d'API, privilégiez un gestionnaire de secrets ou une variable d'environnement plutôt que `environments.ts`. Suivez les bonnes pratiques standard pour sécuriser les clés d'API et les identifiants. Firebase propose désormais un nouveau gestionnaire de secrets avec les dernières mises à jour de Firebase App Hosting. Pour en savoir plus, [consultez la documentation officielle](https://firebase.google.com/docs/app-hosting/configure).

Pour un exemple de connexion côté serveur dans une application full-stack, consultez le code : [dépôt Angular AI Example (Genkit and Angular Story Generator)](https://github.com/angular/examples/tree/main/genkit-angular-story-generator).

### Utiliser le tool calling pour enrichir les applications {#use-tool-calling-to-enhance-apps}

Si vous souhaitez construire des workflows agentiques, dans lesquels des agents peuvent agir et utiliser des outils pour résoudre des problèmes à partir de prompts, utilisez le « tool calling ». Le tool calling, aussi appelé function calling, permet aux LLM d'envoyer des requêtes à l'application qui les a appelés. En tant que développeur, vous définissez quels outils sont disponibles et vous contrôlez comment et quand ils sont appelés.

Le tool calling enrichit encore vos applications web en étendant votre intégration de l'IA au-delà d'un chatbot de type questions-réponses. Vous pouvez en effet permettre à votre modèle de demander des appels de fonctions via l'API de function calling de votre fournisseur de modèles. Les outils disponibles peuvent servir à effectuer des actions plus complexes dans le contexte de votre application.

Dans l'[exemple d'e-commerce](https://github.com/angular/examples/blob/main/firebase-ai-logic-angular-example/src/app/ai.service.ts#L88) du [dépôt d'exemples Angular](https://github.com/angular/examples), le LLM demande à appeler des fonctions liées à l'inventaire afin d'obtenir le contexte nécessaire pour effectuer des tâches plus complexes, comme calculer le coût d'un groupe d'articles du magasin. La portée de l'API disponible dépend de vous, en tant que développeur, tout comme le choix d'appeler ou non une fonction demandée par le LLM. Vous gardez le contrôle du flux d'exécution. Vous pouvez par exemple exposer certaines fonctions d'un service, mais pas toutes.

### Gérer les réponses non déterministes {#handling-non-deterministic-responses}

Comme les modèles peuvent renvoyer des résultats non déterministes, vos applications doivent être conçues en conséquence. Voici quelques stratégies que vous pouvez utiliser dans l'implémentation de votre application :

- Ajustez les prompts et les paramètres du modèle (comme la [température](https://ai.google.dev/gemini-api/docs/prompting-strategies)) pour obtenir des réponses plus ou moins déterministes. Vous pouvez [en savoir plus dans la section sur les stratégies de prompting](https://ai.google.dev/gemini-api/docs/prompting-strategies) de [ai.google.dev](https://ai.google.dev/).
- Utilisez la stratégie « human in the loop », dans laquelle un humain vérifie les sorties avant de poursuivre un workflow. Concevez les workflows de votre application pour permettre à des opérateurs (humains ou autres modèles) de vérifier les sorties et de confirmer les décisions clés.
- Employez le tool calling (ou function calling) et des contraintes de schéma pour guider et restreindre les réponses du modèle à des formats prédéfinis, ce qui rend les réponses plus prévisibles.

Même avec ces stratégies et techniques, la conception de votre application doit intégrer des solutions de repli raisonnables. Suivez les standards existants de résilience des applications. Par exemple, il n'est pas acceptable qu'une application plante si une ressource ou une API n'est pas disponible. Dans ce cas, un message d'erreur est affiché à l'utilisateur et, le cas échéant, des options pour la suite sont également proposées. Construire des applications propulsées par l'IA demande la même attention. Vérifiez que la réponse correspond à la sortie attendue et prévoyez un « atterrissage en douceur » si ce n'est pas le cas, au moyen d'une [dégradation gracieuse](https://developer.mozilla.org/en-US/docs/Glossary/Graceful_degradation). Cela s'applique aussi aux pannes d'API des fournisseurs de LLM.

Prenons cet exemple : le fournisseur de LLM ne répond pas. Une stratégie possible pour gérer la panne est de :

- Sauvegarder la réponse de l'utilisateur pour la réutiliser lors d'une nouvelle tentative (immédiatement ou plus tard)
- Avertir l'utilisateur de la panne avec un message approprié qui ne révèle pas d'informations sensibles
- Reprendre la conversation plus tard, une fois les services de nouveau disponibles.

## Prochaines étapes {#next-steps}

Pour en savoir plus sur les prompts pour LLM et la configuration des IDE IA, consultez les guides suivants :

<docs-pill-row>
  <docs-pill href="ai/develop-with-ai" title="Prompts pour LLM et configuration des IDE"/>
  <docs-pill href="ai/agent-skills" title="Agent Skills"/>
</docs-pill-row>
