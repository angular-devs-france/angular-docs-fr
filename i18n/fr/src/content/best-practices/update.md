<!-- source: 1a7cd0d08367cdcc5d6db1d1a34cd7e82d361eb9 -->
# Garder vos projets Angular à jour

Tout comme le Web et l'ensemble de l'écosystème web, Angular s'améliore en continu.
Angular concilie amélioration continue et forte attention portée à la stabilité et à la simplicité des mises à jour.
Garder votre application Angular à jour vous permet de profiter des nouvelles fonctionnalités de pointe, ainsi que des optimisations et des corrections de bugs.

Ce document contient des informations et des ressources pour vous aider à garder vos applications et libraries Angular à jour.

Pour en savoir plus sur notre politique et nos pratiques de versionnage — notamment en matière de support et de dépréciation, ainsi que sur le calendrier des versions — consultez [Versionnage et versions d'Angular](reference/releases 'Angular versioning and releases').

HELPFUL: Si vous utilisez actuellement AngularJS, consultez [Migrer depuis AngularJS](https://angular.io/guide/upgrade 'Upgrading from AngularJS').
_AngularJS_ est le nom de toutes les versions v1.x d'Angular.

## Être averti des nouvelles versions {#getting-notified-of-new-releases}

Pour être averti de la disponibilité de nouvelles versions, suivez [@angular](https://x.com/angular '@angular on X') sur X (anciennement Twitter) ou abonnez-vous au [blog Angular](https://blog.angular.dev 'Angular blog').

## Découvrir les nouvelles fonctionnalités {#learning-about-new-features}

Quoi de neuf ? Qu'est-ce qui a changé ? Nous partageons l'essentiel de ce que vous devez savoir sur le blog Angular, dans les [annonces de versions](https://blog.angular.dev/ 'Angular blog - release announcements').

Pour consulter la liste complète des changements, organisée par version, consultez le [change log d'Angular](https://github.com/angular/angular/blob/main/CHANGELOG.md 'Angular change log').

## Vérifier votre version d'Angular {#checking-your-version-of-angular}

Pour vérifier la version d'Angular de votre application, utilisez la commande `ng version` depuis le répertoire de votre projet.

## Trouver la version actuelle d'Angular {#finding-the-current-version-of-angular}

La version stable la plus récente d'Angular apparaît [sur npm](https://www.npmjs.com/package/@angular/core 'Angular on npm') sous « Version ». Par exemple, `16.2.4`.

Vous pouvez aussi trouver la version la plus récente d'Angular avec la commande CLI [`ng update`](cli/update).
Par défaut, [`ng update`](cli/update) (sans argument supplémentaire) liste les mises à jour qui vous sont disponibles.

## Mettre à jour votre environnement et vos applications {#updating-your-environment-and-apps}

Pour simplifier les mises à jour, nous fournissons des instructions complètes dans le [guide de mise à jour Angular](update-guide) interactif.

Le guide de mise à jour Angular fournit des instructions de mise à jour personnalisées, selon les versions actuelle et cible que vous indiquez.
Il propose des parcours de mise à jour basiques et avancés, adaptés à la complexité de vos applications.
Il inclut aussi des informations de dépannage et les éventuelles modifications manuelles recommandées pour tirer le meilleur parti de la nouvelle version.

Pour les mises à jour simples, la commande CLI [`ng update`](cli/update) suffit.
Sans argument supplémentaire, [`ng update`](cli/update) liste les mises à jour qui vous sont disponibles et fournit les étapes recommandées pour mettre à jour votre application vers la version la plus récente.

[Versionnage et versions d'Angular](reference/releases#angular-versioning 'Angular Release Practices, Versioning') décrit le niveau de changement auquel vous pouvez vous attendre selon le numéro de version d'une release.
Il décrit aussi les parcours de mise à jour pris en charge.

## Récapitulatif des ressources {#resource-summary}

- Annonces de versions :
  [Blog Angular - annonces de versions](https://blog.angular.dev/ 'Angular blog announcements about recent releases')

- Détails des versions :
  [Change log d'Angular](https://github.com/angular/angular/blob/main/CHANGELOG.md 'Angular change log')

- Instructions de mise à jour :
  [Guide de mise à jour Angular](update-guide)

- Référence de la commande de mise à jour :
  [Référence de la commande Angular CLI `ng update`](cli/update)

- Pratiques de versionnage, de publication, de support et de dépréciation :
  [Versionnage et versions d'Angular](reference/releases 'Angular versioning and releases')
