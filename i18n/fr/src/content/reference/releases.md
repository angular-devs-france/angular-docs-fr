<!-- source: a60ea843aa095af246dc7ae7f3da9d6e89716b74 -->
# Gestion des versions et sorties d'Angular

Nous savons que vous avez besoin de stabilité de la part du framework Angular.
La stabilité garantit que les composants et libraries réutilisables, les tutoriels, les outils et les pratiques acquises ne deviennent pas obsolètes de manière inattendue.
La stabilité est essentielle à l'essor de l'écosystème autour d'Angular.

Nous partageons aussi avec vous le besoin qu'Angular continue d'évoluer.
Nous nous efforçons de faire en sorte que la base sur laquelle vous construisez s'améliore en continu et vous permette de rester à jour avec le reste de l'écosystème web et les besoins de vos utilisateurs.

Ce document présente les pratiques que nous suivons pour vous offrir une plateforme de développement d'applications à la pointe, tout en préservant la stabilité.
Nous nous efforçons de faire en sorte que les changements futurs soient toujours introduits de manière prévisible.
Nous voulons que toute personne qui dépend d'Angular sache quand et comment de nouvelles fonctionnalités sont ajoutées, et soit bien préparée lorsque des fonctionnalités obsolètes sont supprimées.

Il arrive que des _changements incompatibles_ (breaking changes), comme la suppression d'API ou de fonctionnalités, soient nécessaires pour innover et suivre l'évolution des bonnes pratiques, des dépendances ou de la plateforme web. Ces changements incompatibles passent par un processus de dépréciation expliqué dans notre [politique de dépréciation](#deprecation-policy).

Pour rendre ces transitions aussi simples que possible, l'équipe Angular prend les engagements suivants :

- Nous nous efforçons de réduire au minimum le nombre de changements incompatibles et de fournir des outils de migration lorsque c'est possible
- Nous suivons la politique de dépréciation décrite ici, afin que vous ayez le temps de mettre à jour vos applications vers les dernières API et bonnes pratiques

HELPFUL: Les pratiques décrites dans ce document s'appliquent à Angular 2.0 et aux versions ultérieures.
Si vous utilisez actuellement AngularJS, consultez [Migrer depuis AngularJS](https://angular.io/guide/upgrade 'Upgrading from AngularJS').
_AngularJS_ désigne toutes les versions v1.x d'Angular.

## Gestion des versions d'Angular {#angular-versioning}

Les numéros de version d'Angular indiquent le niveau des changements introduits par la version.
Cet usage du [versionnage sémantique](https://semver.org/ 'Semantic Versioning Specification') vous aide à comprendre l'impact potentiel d'une mise à jour vers une nouvelle version.

Les numéros de version d'Angular comportent trois parties : `major.minor.patch`.
Par exemple, la version 7.2.11 indique la version majeure 7, la version mineure 2 et le niveau de patch 11.

Le numéro de version est incrémenté selon le niveau de changement inclus dans la version.

| Niveau de changement | Détails |
| :-------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Version majeure | Contient de nouvelles fonctionnalités importantes ; une intervention des développeurs, minime, est à prévoir lors de la mise à jour. Lors d'une mise à jour vers une nouvelle version majeure, vous pourriez devoir exécuter des scripts de mise à jour, refactoriser du code, exécuter des tests supplémentaires et apprendre de nouvelles API. |
| Version mineure | Contient de nouvelles fonctionnalités plus modestes. Les versions mineures sont entièrement rétrocompatibles ; aucune intervention des développeurs n'est à prévoir lors de la mise à jour, mais vous pouvez, si vous le souhaitez, modifier vos applications et libraries pour commencer à utiliser les nouvelles API, fonctionnalités et capacités ajoutées dans la version. Nous mettons à jour les dépendances peer dans les versions mineures en élargissant les versions prises en charge, mais nous n'exigeons pas des projets qu'ils mettent à jour ces dépendances. |
| Version de patch | Version de correction de bugs, à faible risque. Aucune intervention des développeurs n'est à prévoir lors de la mise à jour. |

HELPFUL: Depuis Angular 7, les versions majeures d'Angular core et de la CLI sont alignées.
Cela signifie que pour utiliser la CLI pendant que vous développez une application Angular, la version de `@angular/core` et celle de la CLI doivent être identiques.

### Préversions {#preview-releases}

Nous vous permettons d'avoir un aperçu de ce qui arrive en publiant des préversions « Next » et Release Candidate \(`rc`\) pour chaque version majeure et mineure :

| Type de préversion | Détails |
| :---------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Next | La version en cours de développement actif et de test. La version next est indiquée par une étiquette de version suffixée par l'identifiant `-next`, comme `8.1.0-next.0`. |
| Release candidate | Une version dont les fonctionnalités sont complètes et qui est en phase de test finale. Une release candidate est indiquée par une étiquette de version suffixée par l'identifiant `-rc`, comme la version `8.1.0-rc.0`. |

La dernière préversion `next` ou `rc` de la documentation est disponible sur [next.angular.dev](https://next.angular.dev).

## Fréquence des sorties {#release-frequency}

Nous visons un calendrier de sorties régulier, afin que vous puissiez planifier et coordonner vos mises à jour avec l'évolution continue d'Angular.

HELPFUL: Les dates sont données à titre indicatif et peuvent changer.

En général, attendez-vous au cycle de sorties suivant :

- Une version majeure tous les 12 mois
- 4 à 6 versions mineures pour chaque version majeure
- Une version de patch et un build de préversion \(`next` ou `rc`\) presque chaque semaine

Ce rythme de sorties donne aux développeurs impatients accès aux nouvelles fonctionnalités dès qu'elles sont entièrement développées et qu'elles ont passé nos processus de revue de code et de tests d'intégration, tout en préservant la stabilité et la fiabilité de la plateforme pour les utilisateurs en production qui préfèrent recevoir les fonctionnalités une fois qu'elles ont été validées par Google et par les autres développeurs qui utilisent les builds de préversion.

HELPFUL: Jusqu'à Angular v22, Angular suivait un cycle de versions majeures de 6 mois, avec 1 à 3 versions mineures pour chaque version majeure

## Politique et calendrier de support {#support-policy-and-schedule}

HELPFUL: Les dates approximatives sont données à titre indicatif et peuvent changer.

### Calendrier des sorties {#release-schedule}

| Version | Date                      |
| :------ | :----------------- |
| v22.1   | Semaine du 27 juillet 2026 |
| v22.2   | Semaine du 21 septembre 2026 |
| v22.3   | ~ novembre 2026    |
| v22.4   | ~ janvier 2027     |
| v22.5   | ~ mars 2027        |
| v23.0   | ~ juin 2027        |

### Période de support {#support-window}

Toutes les versions majeures sont généralement prises en charge pendant 24 mois.

| Phase de support  | Durée          | Détails                                                             |
| :---------------- | :------------- | :------------------------------------------------------------------ |
| Active            | 12 mois        | Des mises à jour et des patchs sont publiés selon un calendrier régulier |
| Long terme \(LTS\) | 12 mois       | Seuls des [correctifs critiques et des patchs de sécurité](#lts-fixes) sont publiés |

### Versions activement prises en charge {#actively-supported-versions}

Le tableau suivant indique le statut des versions d'Angular prises en charge.

| Version | Statut | Sortie     | Fin du support actif | Fin du LTS |
| :------ | :----- | :--------- | :---------- | :--------- |
| ^22.0.0 | Active | 2026-06-03 | 2027-06     | 2028-06    |
| ^21.0.0 | LTS    | 2025-11-19 | 2026-06-03  | 2027-06    |
| ^20.0.0 | LTS    | 2025-05-28 | 2025-11-19  | 2026-11-28 |

IMPORTANT: Les versions v2 à v19 d'Angular ne sont plus prises en charge.

### Correctifs LTS {#lts-fixes}

En règle générale, un correctif est envisagé pour une version LTS s'il résout l'un des problèmes suivants :

- Une faille de sécurité nouvellement identifiée,
- Une régression, apparue depuis le début du LTS, causée par un changement tiers, comme une nouvelle version de navigateur.

## Politique de dépréciation {#deprecation-policy}

Lorsque l'équipe Angular prévoit de supprimer une API ou une fonctionnalité, celle-ci est marquée comme _dépréciée_. Cela se produit lorsqu'une API est obsolète, remplacée par une autre API ou abandonnée pour une autre raison. Les API dépréciées restent disponibles pendant leur phase de dépréciation, qui dure au minimum une version majeure (environ un an).

Pour vous assurer de disposer de suffisamment de temps et d'un chemin de mise à jour clair, voici notre politique de dépréciation :

| Étapes de dépréciation | Détails |
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Annonce | Nous annonçons les API et fonctionnalités dépréciées dans le [changelog](https://github.com/angular/angular/blob/main/CHANGELOG.md 'Angular change log'). Les API dépréciées apparaissent ~~barrées~~ dans la [documentation](api?status=8). Lorsque nous annonçons une dépréciation, nous annonçons aussi un chemin de mise à jour recommandé. De plus, toutes les API dépréciées sont annotées avec `@deprecated` dans la documentation correspondante, ce qui permet aux éditeurs de texte et aux IDE de fournir des indications si votre projet en dépend. |
| Période de dépréciation | Lorsqu'une API ou une fonctionnalité est dépréciée, elle reste présente au moins dans la version majeure suivante (soit une période d'au moins 12 mois). Ensuite, les API et fonctionnalités dépréciées peuvent être supprimées. Une dépréciation peut être annoncée dans n'importe quelle version, mais la suppression d'une API ou d'une fonctionnalité dépréciée n'a lieu que dans une version majeure. Tant qu'une API ou une fonctionnalité dépréciée n'est pas supprimée, elle est maintenue selon la politique de support LTS, ce qui signifie que seuls les problèmes critiques et de sécurité sont corrigés. |
| Dépendances npm | Nous n'effectuons des mises à jour de dépendances npm qui nécessitent des modifications de vos applications que dans une version majeure. Dans les versions mineures, nous mettons à jour les dépendances peer en élargissant les versions prises en charge, mais nous n'exigeons pas des projets qu'ils mettent à jour ces dépendances avant une future version majeure. Cela signifie que, lors des versions mineures d'Angular, les mises à jour de dépendances npm dans les applications et libraries Angular sont facultatives. |

## Politique de compatibilité {#compatibility-policy}

Angular est un ensemble de nombreux packages, sous-projets et outils.
Pour éviter l'utilisation accidentelle d'API privées, et pour que vous compreniez clairement ce qui est couvert par les pratiques décrites ici, nous documentons ce qui fait partie ou non de notre surface d'API publique.
Pour plus de détails, consultez [Surface d'API publique prise en charge d'Angular](https://github.com/angular/angular/blob/main/contributing-docs/public-api-surface.md 'Supported Public API Surface of Angular').

Pour garantir la rétrocompatibilité d'Angular, nous effectuons une série de vérifications avant de fusionner tout changement :

- Tests unitaires et tests d'intégration
- Comparaison des définitions de types de la surface d'API publique avant et après le changement
- Exécution des tests de toutes les applications de Google qui dépendent d'Angular

Tout changement de la surface d'API publique est effectué conformément aux politiques de gestion des versions, de support et de dépréciation décrites précédemment. Dans des cas exceptionnels, comme des patchs de sécurité critiques, des correctifs peuvent introduire des changements non rétrocompatibles. Ces cas exceptionnels sont accompagnés d'un avis explicite sur les canaux de communication officiels du framework.

## Politique des changements incompatibles et chemins de mise à jour {#breaking-change-policy-and-update-paths}

Un changement incompatible vous demande du travail, car l'état qui en résulte n'est pas rétrocompatible avec l'état antérieur. Vous trouverez les rares exceptions à cette règle dans la [politique de compatibilité](#compatibility-policy). Parmi les exemples de changements incompatibles figurent la suppression d'API publiques ou d'autres modifications des définitions de types d'Angular, la modification du moment des appels, ou la mise à jour vers une nouvelle version d'une dépendance d'Angular qui comporte elle-même des changements incompatibles.

Pour vous accompagner en cas de changements incompatibles dans Angular :

- Nous suivons notre [politique de dépréciation](#deprecation-policy) avant de supprimer une API publique
- Nous prenons en charge l'automatisation des mises à jour via la commande `ng update`. Elle fournit des transformations de code que nous avons souvent testées à l'avance sur des centaines de milliers de projets chez Google
- Des instructions étape par étape pour passer d'une version majeure à une autre sont disponibles dans le [« guide de mise à jour Angular »](update-guide)

Vous pouvez utiliser `ng update` vers n'importe quelle version d'Angular, à condition que les critères suivants soient remplis :

- La version _vers_ laquelle vous voulez mettre à jour est prise en charge.
- La version _depuis_ laquelle vous voulez mettre à jour se situe à une version majeure au plus de la version vers laquelle vous voulez
  migrer.

Par exemple, vous pouvez mettre à jour de la version 11 vers la version 12, à condition que la version 12 soit toujours prise en charge.
Si vous voulez franchir plusieurs versions majeures, effectuez chaque mise à jour une version majeure à la fois.
Par exemple, pour mettre à jour de la version 10 vers la version 12 :

1. Mettez à jour de la version 10 vers la version 11.
1. Mettez à jour de la version 11 vers la version 12.

## Developer Preview {#developer-preview}

Nous introduisons parfois de nouvelles API sous le label « Developer Preview ». Ce sont des API entièrement fonctionnelles et abouties, mais que nous ne sommes pas encore prêts à stabiliser dans le cadre de notre politique de dépréciation habituelle.

Cela peut être parce que nous voulons recueillir des retours d'applications réelles avant la stabilisation, ou parce que la documentation ou l'outillage de migration associés ne sont pas entièrement terminés. Vous pouvez faire vos retours via une [issue GitHub](https://github.com/angular/angular/issues), où les développeurs peuvent partager leurs expériences, signaler des bugs ou suggérer des améliorations pour aider à affiner la fonctionnalité.

Les politiques et pratiques décrites dans ce document ne s'appliquent pas aux API marquées comme Developer Preview. Ces API peuvent changer à tout moment, même dans de nouvelles versions de patch du framework. Les équipes doivent décider elles-mêmes si les avantages de l'utilisation d'API en Developer Preview valent le risque de changements incompatibles en dehors de notre usage habituel du versionnage sémantique.

## Expérimental {#experimental}

Ces API pourraient ne jamais devenir stables ou subir des changements importants avant de le devenir.

Les politiques et pratiques décrites dans ce document ne s'appliquent pas aux API marquées comme expérimentales. Ces API peuvent changer à tout moment, même dans de nouvelles versions de patch du framework. Les équipes doivent décider elles-mêmes si les avantages de l'utilisation d'API expérimentales valent le risque de changements incompatibles en dehors de notre usage habituel du versionnage sémantique.
