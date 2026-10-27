<!-- source: e273fb2166a42cb65d156108387d4620fdbbe474 -->
# Compatibilité des versions

Les tableaux suivants décrivent les versions de Node.js, TypeScript et RxJS requises par chaque version
d'Angular.

## Versions activement prises en charge {#actively-supported-versions}

Ce tableau couvre les [versions d'Angular activement prises en charge](reference/releases#actively-supported-versions).

| Angular                        | Node.js                             | TypeScript     | RxJS               |
| ------------------------------ | ----------------------------------- | -------------- | ------------------ |
| 22.0.x                         | ^22.22.3 \|\| ^24.15.0 \|\| ^26.0.0 | >=6.0.0 <6.1.0 | ^6.5.3 \|\| ^7.4.0 |
| 21.0.x \|\| 21.1.x \|\| 21.2.x | ^20.19.0 \|\| ^22.12.0 \|\| ^24.0.0 | >=5.9.0 <6.0.0 | ^6.5.3 \|\| ^7.4.0 |
| 20.2.x \|\| 20.3.x             | ^20.19.0 \|\| ^22.12.0 \|\| ^24.0.0 | >=5.8.0 <6.0.0 | ^6.5.3 \|\| ^7.4.0 |
| 20.0.x \|\| 20.1.x             | ^20.19.0 \|\| ^22.12.0 \|\| ^24.0.0 | >=5.8.0 <5.9.0 | ^6.5.3 \|\| ^7.4.0 |

## Versions d'Angular non prises en charge {#unsupported-angular-versions}

Ce tableau couvre les versions d'Angular qui ne bénéficient plus du support à long terme (LTS). Ces
informations étaient exactes lorsque chaque version est sortie du LTS et sont fournies sans autre
garantie. Elles sont listées ici à titre de référence historique.

| Angular            | Node.js                              | TypeScript     | RxJS               |
| ------------------ | ------------------------------------ | -------------- | ------------------ |
| 19.2.x             | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0.0  | >=5.5.0 <5.9.0 | ^6.5.3 \|\| ^7.4.0 |
| 19.1.x             | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0.0  | >=5.5.0 <5.8.0 | ^6.5.3 \|\| ^7.4.0 |
| 19.0.x             | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0.0  | >=5.5.0 <5.7.0 | ^6.5.3 \|\| ^7.4.0 |
| 18.1.x \|\| 18.2.x | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0.0  | >=5.4.0 <5.6.0 | ^6.5.3 \|\| ^7.4.0 |
| 18.0.x             | ^18.19.1 \|\| ^20.11.1 \|\| ^22.0.0  | >=5.4.0 <5.5.0 | ^6.5.3 \|\| ^7.4.0 |
| 17.3.x             | ^18.13.0 \|\| ^20.9.0                | >=5.2.0 <5.5.0 | ^6.5.3 \|\| ^7.4.0 |
| 17.1.x \|\| 17.2.x | ^18.13.0 \|\| ^20.9.0                | >=5.2.0 <5.4.0 | ^6.5.3 \|\| ^7.4.0 |
| 17.0.x             | ^18.13.0 \|\| ^20.9.0                | >=5.2.0 <5.3.0 | ^6.5.3 \|\| ^7.4.0 |
| 16.1.x \|\| 16.2.x | ^16.14.0 \|\| ^18.10.0               | >=4.9.3 <5.2.0 | ^6.5.3 \|\| ^7.4.0 |
| 16.0.x             | ^16.14.0 \|\| ^18.10.0               | >=4.9.3 <5.1.0 | ^6.5.3 \|\| ^7.4.0 |
| 15.1.x \|\| 15.2.x | ^14.20.0 \|\| ^16.13.0 \|\| ^18.10.0 | >=4.8.2 <5.0.0 | ^6.5.3 \|\| ^7.4.0 |
| 15.0.x             | ^14.20.0 \|\| ^16.13.0 \|\| ^18.10.0 | ~4.8.2         | ^6.5.3 \|\| ^7.4.0 |
| 14.2.x \|\| 14.3.x | ^14.15.0 \|\| ^16.10.0               | >=4.6.2 <4.9.0 | ^6.5.3 \|\| ^7.4.0 |
| 14.0.x \|\| 14.1.x | ^14.15.0 \|\| ^16.10.0               | >=4.6.2 <4.8.0 | ^6.5.3 \|\| ^7.4.0 |
| 13.3.x \|\| 13.4.x | ^12.20.0 \|\| ^14.15.0 \|\| ^16.10.0 | >=4.4.3 <4.7.0 | ^6.5.3 \|\| ^7.4.0 |
| 13.1.x \|\| 13.2.x | ^12.20.0 \|\| ^14.15.0 \|\| ^16.10.0 | >=4.4.3 <4.6.0 | ^6.5.3 \|\| ^7.4.0 |
| 13.0.x             | ^12.20.0 \|\| ^14.15.0 \|\| ^16.10.0 | ~4.4.3         | ^6.5.3 \|\| ^7.4.0 |
| 12.2.x             | ^12.14.0 \|\| ^14.15.0               | >=4.2.3 <4.4.0 | ^6.5.3 \|\| ^7.0.0 |
| 12.1.x             | ^12.14.0 \|\| ^14.15.0               | >=4.2.3 <4.4.0 | ^6.5.3             |
| 12.0.x             | ^12.14.0 \|\| ^14.15.0               | ~4.2.3         | ^6.5.3             |
| 11.2.x             | ^10.13.0 \|\| ^12.11.0               | >=4.0.0 <4.2.0 | ^6.5.3             |
| 11.1.x             | ^10.13.0 \|\| ^12.11.0               | >=4.0.0 <4.2.0 | ^6.5.3             |
| 11.0.x             | ^10.13.0 \|\| ^12.11.0               | ~4.0.0         | ^6.5.3             |
| 10.2.x             | ^10.13.0 \|\| ^12.11.0               | >=3.9.0 <4.1.0 | ^6.5.3             |
| 10.1.x             | ^10.13.0 \|\| ^12.11.0               | >=3.9.0 <4.1.0 | ^6.5.3             |
| 10.0.x             | ^10.13.0 \|\| ^12.11.0               | ~3.9.0         | ^6.5.3             |
| 9.1.x              | ^10.13.0 \|\| ^12.11.0               | >=3.6.0 <3.9.0 | ^6.5.3             |
| 9.0.x              | ^10.13.0 \|\| ^12.11.0               | >=3.6.0 <3.8.0 | ^6.5.3             |

### Avant la v9 {#before-v9}

Jusqu'à Angular v9, les versions d'Angular et de l'Angular CLI n'étaient pas synchronisées.

| Angular                     | Angular CLI                 | Node.js             | TypeScript     | RxJS   |
| --------------------------- | --------------------------- | ------------------- | -------------- | ------ |
| 8.2.x                       | 8.2.x \|\| 8.3.x            | ^10.9.0             | >=3.4.2 <3.6.0 | ^6.4.0 |
| 8.0.x \|\| 8.1.x            | 8.0.x \|\| 8.1.x            | ^10.9.0             | ~3.4.2         | ^6.4.0 |
| 7.2.x                       | 7.2.x \|\| 7.3.x            | ^8.9.0 \|\| ^10.9.0 | >=3.1.3 <3.3.0 | ^6.0.0 |
| 7.0.x \|\| 7.1.x            | 7.0.x \|\| 7.1.x            | ^8.9.0 \|\| ^10.9.0 | ~3.1.3         | ^6.0.0 |
| 6.1.x                       | 6.1.x \|\| 6.2.x            | ^8.9.0              | >=2.7.2 <3.0.0 | ^6.0.0 |
| 6.0.x                       | 6.0.x                       | ^8.9.0              | ~2.7.2         | ^6.0.0 |
| 5.2.x                       | 1.6.x \|\| 1.7.x            | ^6.9.0 \|\| ^8.9.0  | >=2.4.2 <2.7.0 | ^5.5.0 |
| 5.0.x \|\| 5.1.x            | 1.5.x                       | ^6.9.0 \|\| ^8.9.0  | ~2.4.2         | ^5.5.0 |
| 4.2.x \|\| 4.3.x \|\| 4.4.x | 1.4.x                       | ^6.9.0 \|\| ^8.9.0  | >=2.1.6 <2.5.0 | ^5.0.1 |
| 4.2.x \|\| 4.3.x \|\| 4.4.x | 1.3.x                       | ^6.9.0              | >=2.1.6 <2.5.0 | ^5.0.1 |
| 4.0.x \|\| 4.1.x            | 1.0.x \|\| 1.1.x \|\| 1.2.x | ^6.9.0              | >=2.1.6 <2.4.0 | ^5.0.1 |
| 2.x                         | -                           | ^6.9.0              | >=1.8.0 <2.2.0 | ^5.0.1 |

## Prise en charge des navigateurs {#browser-support}

Angular utilise la [Baseline « widely available »](https://web.dev/baseline) pour définir les navigateurs
pris en charge. Pour chaque version majeure, Angular prend en charge les navigateurs inclus dans la Baseline d'une
date choisie proche de la date de sortie de cette version majeure.

La Baseline « widely available » inclut les navigateurs sortis moins de 30 mois (2,5 ans)
avant la date choisie parmi l'ensemble de navigateurs de référence de Baseline (Chrome, Edge, Firefox, Safari) et
vise à couvrir environ 95 % des utilisateurs du web.

| Angular | Date Baseline | Navigateurs                 |
| ------- | ------------- | --------------------------- |
| v22     | 2026-05-07    | [Navigateurs][browsers-v22] |
| v21     | 2025-10-20    | [Navigateurs][browsers-v21] |
| v20     | 2025-04-30    | [Navigateurs][browsers-v20] |

[browsers-v22]: https://web-platform-dx.github.io/supported-browsers/?widelyAvailableOnDate=2026-05-07&includeDownstream=false
[browsers-v21]: https://web-platform-dx.github.io/supported-browsers/?widelyAvailableOnDate=2025-10-20&includeDownstream=false
[browsers-v20]: https://web-platform-dx.github.io/supported-browsers/?widelyAvailableOnDate=2025-04-30&includeDownstream=false

Les versions d'Angular antérieures à la v20 prennent en charge les versions de navigateurs suivantes :

| Navigateur | Versions prises en charge                 |
| :------ | :------------------------------------------ |
| Chrome  | Les 2 versions les plus récentes            |
| Firefox | Dernière version et version à support étendu \(ESR\) |
| Edge    | Les 2 versions majeures les plus récentes   |
| Safari  | Les 2 versions majeures les plus récentes   |
| iOS     | Les 2 versions majeures les plus récentes   |
| Android | Les 2 versions majeures les plus récentes   |

## Polyfills {#polyfills}

Angular repose sur les derniers standards de la plateforme web.
Cibler un éventail aussi large de navigateurs est difficile, car ils ne prennent pas en charge toutes les fonctionnalités des navigateurs modernes.
Vous compensez cela en chargeant des scripts de polyfill \(« polyfills »\) pour les navigateurs que vous devez prendre en charge.
Consultez ci-dessous les instructions pour inclure des polyfills dans votre projet.

IMPORTANT: Les polyfills suggérés sont ceux qui permettent d'exécuter des applications Angular complètes.
Vous pourriez avoir besoin de polyfills supplémentaires pour prendre en charge des fonctionnalités non couvertes par cette liste.

HELPFUL: Les polyfills ne peuvent pas transformer comme par magie un navigateur ancien et lent en un navigateur moderne et rapide.

## Activer les polyfills dans les projets CLI {#enabling-polyfills-with-cli-projects}

L'[Angular CLI](tools/cli) prend en charge les polyfills.
Si vous n'utilisez pas la CLI pour créer vos projets, consultez les [instructions sur les polyfills pour les utilisateurs sans CLI](#polyfills-for-non-cli-users).

Les options `polyfills` du [builder browser et du builder de test](tools/cli/cli-builder) peuvent être un chemin complet vers un fichier \(exemple : `src/polyfills.ts`\) ou
un chemin relatif au workspace courant, ou encore un spécificateur de module \(exemple : `zone.js`\).

Si vous créez un fichier TypeScript, veillez à l'inclure dans la propriété `files` de votre fichier `tsconfig`.

```json

{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    ...
  },
  "files": [
    "src/main.ts",
    "src/polyfills.ts"
  ]
  ...
}
```

## Polyfills pour les utilisateurs sans CLI {#polyfills-for-non-cli-users}

Si vous n'utilisez pas la CLI, ajoutez vos scripts de polyfill directement dans la page web hôte \(`index.html`\).

Par exemple :

<docs-code header="src/index.html" language="html">
<!-- pre-zone polyfills -->
<script src="node_modules/core-js/client/shim.min.js"></script>
<script>
  /**
   * you can configure some zone flags which can disable zone interception for some
   * asynchronous activities to improve startup performance - use these options only
   * if you know what you are doing as it could result in hard to trace down bugs.
   */
  // __Zone_disable_requestAnimationFrame = true; // disable patch requestAnimationFrame
  // __Zone_disable_on_property = true; // disable patch onProperty such as onclick
  // __zone_symbol__UNPATCHED_EVENTS = ['scroll', 'mousemove']; // disable patch specified eventNames
  /*
   * in Edge developer tools, the addEventListener will also be wrapped by zone.js
   * with the following flag, it will bypass `zone.js` patch for Edge.
   */
  // __Zone_enable_cross_context_check = true;
</script>
<!-- zone.js required by Angular -->
<script src="node_modules/zone.js/bundles/zone.umd.js"></script>
<!-- application polyfills -->
</docs-code>
