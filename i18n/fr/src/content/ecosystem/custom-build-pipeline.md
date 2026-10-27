<!-- source: d2bab6eba505d8efa35d88321ddc618889f05ba2 -->
# Pipeline de build personnalisé

Pour compiler une application Angular, nous vous recommandons vivement d'utiliser l'Angular CLI afin de profiter de ses fonctionnalités de mise à jour adaptées à la structure du projet et de l'abstraction de son système de build. Vos projets bénéficient ainsi des dernières améliorations en matière de sécurité, de performances et d'API, ainsi que d'améliorations transparentes du build.

Cette page présente les **rares cas d'usage** où vous avez besoin d'un pipeline de build personnalisé qui n'utilise pas l'Angular CLI. Tous les outils listés ci-dessous sont des plugins de build open source maintenus par des membres de la communauté Angular. Pour en savoir plus sur leur modèle de support et leur état de maintenance, consultez leur documentation et les URL de leurs dépôts GitHub.

## Quand utiliser un pipeline de build personnalisé ? {#when-should-you-use-a-custom-build-pipeline}

Il existe quelques cas d'usage de niche dans lesquels vous pouvez vouloir maintenir un pipeline de build personnalisé. Par exemple :

- Vous avez une application existante qui utilise une autre chaîne d'outils et vous souhaitez y ajouter Angular
- Vous êtes fortement lié à la [module federation](https://module-federation.io/) et ne pouvez pas adopter la [native federation](https://www.npmjs.com/package/@angular-architects/native-federation), indépendante du bundler
- Vous souhaitez mener une expérimentation de courte durée avec votre outil de build préféré

## Quelles sont les options ? {#what-are-the-options}

Il existe actuellement deux outils communautaires bien maintenus qui vous permettent de créer un pipeline de build personnalisé, avec un [plugin Vite](https://www.npmjs.com/package/@analogjs/vite-plugin-angular) et un [plugin Rspack](https://www.npmjs.com/package/@nx/angular-rspack). Tous deux utilisent les abstractions sous-jacentes sur lesquelles repose l'Angular CLI. Ils vous permettent de créer un pipeline de build flexible, mais nécessitent une maintenance manuelle et n'offrent pas d'expérience de mise à jour automatisée.

### Rspack {#rspack}

Rspack est un bundler écrit en Rust qui vise à offrir une compatibilité avec l'écosystème de plugins webpack.

Si votre projet est étroitement lié à l'écosystème webpack et repose fortement sur une configuration webpack personnalisée, vous pouvez tirer parti de Rspack pour améliorer vos temps de build.

Vous trouverez plus d'informations sur Angular Rspack sur le [site de documentation](https://nx.dev/recipes/angular/rspack/introduction) du projet.

### Vite {#vite}

Vite est un outil de build frontend qui vise à offrir une expérience de développement plus rapide et plus légère pour les projets web modernes. Vite est également extensible grâce à son système de plugins, qui permet aux écosystèmes de créer des intégrations avec Vite, comme Vitest pour les tests unitaires et les tests dans le navigateur, Storybook pour concevoir des composants de manière isolée, et bien d'autres. L'Angular CLI utilise également Vite comme serveur de développement.

Le [plugin Vite AnalogJS pour Angular](https://www.npmjs.com/package/@analogjs/vite-plugin-angular) permet d'adopter Angular dans un projet ou un framework qui utilise Vite ou qui repose sur Vite. Il peut s'agir de développer et de compiler un projet Angular directement avec Vite, ou d'ajouter Angular à un projet ou à un pipeline existant. Un exemple consiste à intégrer des composants d'interface Angular dans un site de documentation à l'aide d'[Astro et Starlight](https://analogjs.org/docs/packages/astro-angular/overview).

Vous pouvez en savoir plus sur AnalogJS et sur l'utilisation du plugin sur sa [page de documentation](https://analogjs.org/docs/packages/vite-plugin-angular/overview).
